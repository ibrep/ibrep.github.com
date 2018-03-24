---
layout: post
title:  "PMON start.S源码分析"
date:   2018-03-24 13:33:25 +0800
categories: boot
---
## PMON 的启动流程
```
	.set	noreorder
	.globl	_start
	.globl	start
	.globl	__main
_start:
start:
```
定义了一堆符号，几乎把可能有用的都定义了。

```
    .globl	stack
stack = start - 0x4000		/* Place PMON stack below PMON start in RAM */
```
`start.o`有两份，现在运行的这个是连接到`gzrom`中的，其连接地址为`0x8f90 0000`，因此`start`也应该是这个值。
计算出的堆栈地址为`0x8f900000-0x4000=0x8F8FC000`。这个位置位于内存中最低256M中，249M左右的位置。

```
	/*set all spi cs to 1, default input*/
	li v0,0xbfff0225        /* 这是SPI片选控制寄存器的地址 */
	li v1,0xff              /* 全部置为1，使能全部四个片选信号线 */
	sb v1,(v0)
```
接下来这段是修复硬件bug??? 有些寄存器不知道干什么用的，手册上没写，*Undocumented Loongson!*
```
#if 1 //fix the hardware poweroff error.

	.set    mips32
	mfc0    t0, $15, 1      # EBASE /* 读取系统引导时的异常向量基地址 */
	.set    mips3
	andi    t0, t0, 0x3ff   /* EBASE[11:0]中为CPU编号，多CPU系统中用来区分是哪一个 */
	bnez    t0, 2f          /* 不是0号CPU，则结束 */
	nop

	lui	t0, 0xba00          /* 0xba001010 是什么鬼？*/
	lui	t1, 0x1fe0
	sw	t1, 0x1010(t0) /* config bar for APB */
	lw	t2, 0x1004(t0)      /* 0xba001004 又是什么鬼？*/
	ori	t2, t2, 0x2
	sw	t2, 0x1004(t0)

	li t0,0xbfe0700c
	lw t1,0x0(t0)
	and t2,t1,(1 << 11)
	beqz  t2,2f
	nop

	li t0,0xbfe0700c
	lw t1, 0x0(t0)
	sw t1,0x0(t0)
	li t2,0x3c00
	li t0,0xbfe07014
	sw t2,0x0(t0)
2:
#endif
```
接着初始化处理器状态、初始化寄存器、设置sp和gp：

```
/* init processor state at first*/
...
    bal     initregs   /* 所有寄存器清零; 初始化浮点CP1; 设置 sp=stack, gp=_gp */
    nop
```
*Undocumented Loongson!*又来了，CP0 Register 16, select 6 is reserved for implementation depended use, 干嘛用的？龙芯的手册中并没有说明。
```
	.set	mips32
	mfc0	t0, $16, 6		#Store fill
	.set	mips3
	li	t1, 0xfffffeff
	and	t0, t1, t0
	.set	mips32
	mtc0	t0, $16, 6		#Store fill
	.set	mips3
```
然后：
```
/* spi speed up */
...
bal locate /* Get current execute address */
```
接下来的动作：
  1. `locate`计算运行地址与连接地址的差值，并将差值存入s0;
  2. enable 64bit space;
  3. 读取Ebase中的CPU编号，判断自己是否为CPU0;
  4. `bal beep_on`
  5. 延时;
  6. `bal beep_off`
  7. config bar for APB; *Undocumented Loongson*
  8. config pcie; setup pcie1 port0, port1, port2, port3;
  9. `bal initserial`: 初始化串口;
  10. 打印`initserial good ^_^...`
  11. `#include "loongson3_clksetting.S"`设置PLL时钟;
  12. `bal initserial_later`: 再次设置串口，波特率设置为最大值
  13. Config SATA; 打印`USE internel SATA ref clock`
  14. Fix the Gmac0  multi-func to enable Gmac1;
  15. Set the invalid BAR to read only;
  16. start_now: 打印`PMON2000 MIPS Initializing. Standby...`
  17. enable kseg0 cachability; 启用kseg0缓存
  18. 从启动时的地址`0xbfc.....`跳转到`0x9fc.....`; 虽然仍在flash上运行，但有了cache，速度会快一些
  19. 设置 pcie0 和 pcie1
  20. enable dvo0 and dvo1 pin output;
  21. enable pwm0, pwm1, i2c0, i2c1, nand, sata, i2s, gmac1; no hda, no ac97
  22. 打印`Start Init Memory, wait a while......`
  23. `#include "ddr_dir/loongson3_ddr2_config.S"`设置DDR3内存
  24. test memory
  25. 打印`The uncache data is:`
  26. 打印`The cached data is:`
  27. `#include "machine/newtest/newdebug.S"`
  28. 打印`start=0x... s0=0x... _edata=0x... _end=0x...`
  29. 将代码段从flash rom(`0xbfc00000`)中拷贝到内存中的`0x8f900000`，这个位置是连接生成gzrom目标文件时指定的;
  30. Clear BSS;
  31. 打印`Copy PMON to execute location done.`;
  32. 跳转到`initmips()`; 注意这是那个假的`initmips()`，源码在`zloader/initmips.c`中

