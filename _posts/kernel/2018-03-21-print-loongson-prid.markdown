---
layout: post
title:  "打印龙芯的处理器版本号"
date:   2018-03-21 18:58:25 +0800
categories: kernel
---
# 查看龙芯的处理器版本号
## 版本号寄存器PRID
MIPS CPU中的Processor Revision Identifier (PRID)寄存器中包含了处理器的版本信息。

根据See Mips Run所讲，这个寄存器的各个域分别表示：
  1. (31:24) Company Options
  2. (23:16) Company ID
  3. (15:8) Processor ID
  4. (7:0)  Revision

## 打印PRID的值
打印PRID的值需要使用只能在核心态使用的特权指令`mfc0`，因此只能编写内核模块。
```C
[brep@Loongson]$ cat prid.c 
#include <linux/init.h>
#include <linux/module.h>
MODULE_LICENSE("GPL");

static int prid_init(void)
{
	int prid;
	asm (
		"mfc0 %[result], $15, 0\n"
		: [result] "=r" (prid)
	);
	printk(KERN_ALERT "prid=0x%X\n", prid);
	return 0;
}

static void prid_exit(void)
{
	printk(KERN_ALERT "prid exit\n");
}

module_init(prid_init);
module_exit(prid_exit);
```

```C
[brep@Loongson]$ cat Makefile 
MODDIR ?= /lib/modules/$(shell uname -r)/build
PWD := $(shell pwd)

obj-m += prid.o

modules:
	$(MAKE) -C $(MODDIR) M=$(PWD) modules

clean:
	rm -rf *.o *.ko *.cmd *.mod.* Module.symvers modules.order

.PHONY: modules

```

```C
[brep@Loongson]$ make
[brep@Loongson]$ sudo insmod prid.ko
[brep@Loongson]$ sudo rmmod prid.ko
[brep@Loongson]$ sudo dmesg | tail -n2
[24182.679687] prid=0x146309
[24218.792968] prid exit
[brep@Loongson]$
```

## 龙芯3A3000的处理器版本号
由输出结果可以看出，笔者所用的龙芯3A3000 CPU的版本号为：
  1. Company Options = 0
  2. Company ID = 0x14
  3. Processor ID = 0x63
  4. Revision = 0x09
  
