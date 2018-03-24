---
layout: post
title:  "PMON start.S 中的几个子程序"
date:   2018-03-24 12:24:25 +0800
categories: boot
---
# 通过串口输出一个字符
调用前，先把需要打印的字符写入a0寄存器。
```
LEAF(tgt_putchar)
	.set noat                /* 通知汇编器，我要用at，你就别用了 */
	move	AT, ra
	la	v0, COM1_BASE_ADDR   /* 串口1的基地址 */
	bal	1f
	nop
	jr	AT
	nop
  
1:
	lbu	v1, NSREG(NS16550_LSR)(v0)   /* 读串口状态寄存器 */
	and	v1, LSR_TXRDY                /* 如果忙，跳回去再读 */
	beqz	v1, 1b
	nop
						 
	sb	a0, NSREG(NS16550_DATA)(v0)  /* 将a0中的字符写入串口数据寄存器 */
	j	ra
	nop
	.set at                          /* 我不用at了，你汇编器可以用了 */
END(tgt_putchar
```
# 通过串口打印字符串
调用前，先把需要打印的以`\0`结尾的字符串的地址写入a0寄存器。
```
LEAF(stringserial)
	move	a2, ra      /* 将返回地址保存在a2中 */
	addu	a1, a0, s0  /* 调整字符串地址，s0中是计算出的运行地址与链接地址的差值 */
	lbu	a0, 0(a1)       /* 装入一个字符到a0 */
1:
	beqz	a0, 2f      /* 遇到结尾的0，结束 */
	nop
	bal	tgt_putchar     /* 调用打印单个字符的子程序 */
	addiu	a1, 1       /* a1=下一个字符的地址 */
	b	1b
	lbu	a0, 0(a1)       /* 延迟槽，装入下一个字符到a0 */
2:
	j	a2
	nop
END(stringserial)
```
## 打印字符串的宏
```
#define	PRINTSTR(x) \
	.rdata;98: .asciz x; .text; la a0, 98b; bal stringserial; nop
    
#define	TTYDBG(x) \
	.rdata;98: .asciz x; .text; la a0, 98b; bal stringserial; nop
```
这两个宏都使用了`stringserial`子程序。


