---
layout: post
title:  "PMON initmips()源码分析"
date:   2018-03-25 12:33:25 +0800
categories: boot
---
# 假`initmips()`源码分析
PMON的启动代码`start.S`执行完成之后，跳转到了`zloader/initmips.c`中的`initmips()`函数继续执行。

`zloader/initmips.c`并不是手工编写的源代码文件，而是由一个叫做`genrom`的`perl`脚本生成的。
这个脚本的主要任务是打开目标代码文件`pmon`，在其中搜索以下几个符号的地址：
  1. `_edata`: 数据段的结尾地址，也是`bss`开始地址
  2. `_end_`: `bss`段的结尾地址
  3. `initmips`: 真正的`initmips()`函数的入口地址，这个函数是在`Targets/LS2K/ls2k/tgt_machdep.c`中定义的。
  4. `_start_`: 入口地址，但是因为`start.o`已经执行过一遍了，所以得到这个地址并不是为了再次执行`start.o`，而是
  为了确定要将`biosdata[]`解压到何处，以及确定堆栈的起始地址，因为`stack=_start - 0x4000`。
  
现在分析`initmips()`的执行流程：
  1. 打印`Uncompressing Bios`;
  2. `run_unzip(biosdata, _start)`: 解压缩`biosdata[]`到地址`_start`;
  3. 打印"OK, Booting Bios";
  4. `bss`清零，也就是从`_edata`到`_end`这一段内存;
  5. `_start-0x1000`到`_start`这一段内存清零;
  6. 跳到`realinitmips()`执行;
  7. 在`realinitmips()`中，设置堆栈起点为 `_start - 0x4000`, 跳转到真正的`initmips()`执行。
  
# 真`initmips()`源码分析
真正的`initmips()`函数是在`Targets/LS2K/ls2k/tgt_machdep.c`中定义的。

现在分析真正的`initmips()`的执行流程：
  1. 启动CPU1？*Undocumented Loongson!*
  2. 设置`sata gpio`???
  3. 使能FPU;
  4. 清TLB;
  5. 设置内存边界;
  6. 探测CPU工作频率，打印`cpu freq %u\n`;
  7. `CPU_ConfigCache()`: 在`pmon/arch/mips/cache.S`中定义;
  8. spi, nand, gpio, pwm 相关设置;
  9. main()
