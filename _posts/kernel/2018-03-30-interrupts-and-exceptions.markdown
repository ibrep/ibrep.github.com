---
layout: post
title:  "MIPS的中断和异常"
date:   2018-03-30 18:15:25 +0800
categories: kernel
---
## 异常向量的位置
Reset, Soft Reset和NMI异常的入口地址始终是 `0xBFC0 0000`;

EJTAG Debug异常的入口地址是 `0xBFC0 0480 (if ProbTrap=0)`, 或者`0xFF20 0200 (if ProbTrap=1)`;


## EBase
CP0 Ebase寄存器中保存着异常向量的基地址，系统复位是Ebase的值是`0x8000 0000`。这个值是可以修改的，
也就是允许用户指定异常向量的位置。但PMON和Linux内核中都没有修改这个值，因此`EBase=0x80000000`。

## 异常向量偏移量

异常      | 向量偏移
---------------------
TLB Refill, EXL=0 | 0x000
64-bit XTLB Refill, EXL=0 | 0x080
Cache error | 0x100
General Exception | 0x180
Interrupt, Cause_IV=0 | 0x180
Interrupt, Cause_IV=1 | 0x200

注意中断的偏移量受`Cause_IV`的影响

## 中断的初始化过程
```
unsigned long exception_handlers[32];

start_kernel()
    trap_init()
	    set_handler(0x180, &except_vec3_generic, 0x80);
		    set_except_vector(int n, void *addr);
			    exception_handlers[n] = addr;
```
默认的异常处理函数为`handle_reserved`:
```
	/*
	 * Setup default vectors
	 */
	for (i = 0; i <= 31; i++)
		set_except_vector(i, handle_reserved);
```

## 异常处理函数
```
extern asmlinkage void handle_adel(void);
extern asmlinkage void handle_ades(void);
extern asmlinkage void handle_ibe(void);
extern asmlinkage void handle_dbe(void);
extern asmlinkage void handle_sys(void);
extern asmlinkage void handle_bp(void);
extern asmlinkage void handle_ri(void);
extern asmlinkage void handle_ri_rdhwr_vivt(void);
extern asmlinkage void handle_ri_rdhwr(void);
extern asmlinkage void handle_cpu(void);
extern asmlinkage void handle_ov(void);
extern asmlinkage void handle_tr(void);
extern asmlinkage void handle_fpe(void);
extern asmlinkage void handle_ftlb(void);
extern asmlinkage void handle_mdmx(void);
extern asmlinkage void handle_watch(void);
extern asmlinkage void handle_mt(void);
extern asmlinkage void handle_dsp(void);
extern asmlinkage void handle_mcheck(void);
extern asmlinkage void handle_reserved(void);
```
这些函数都是在 `arch/mips/kernel/genex.S`文件中用`BUILD_HANDLER`宏造出来的。
