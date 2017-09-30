---
layout: post
title:  "2. 寄存器操作"
date: 2017-09-25 00:49:25 +0800
categories: mips-asm
order: 2
---
<a href="https://www.bilibili.com/video/av14809674/" target="_blank"><img src="{{ site.url }}/assets/images/video-icon.png" alt="龙芯汇编语言程序设计教程" /></a>

# 本节课的目标
  * 了解龙芯CPU中的寄存器
  * 寄存器操作指令move
  * hi/lo寄存器操作指令

# 电脑是如何工作的
<img src="{{ site.url }}/assets/images/mips-asm/computer-model.png" />

# 龙芯CPU中的寄存器
<img src="{{ site.url }}/assets/images/mips-asm/mips64-registers.png" />

# 查看寄存器中的内容
用 GDB (GNU Debugger) 调试器可以查看寄存器的内容：

	$ gdb add

常用gdb命令：

| 命令                | 作用           | 简化记号  |
|---------------------|----------------|-----------|
| start               | 启动程序       |           |
| stepi               | 单步执行       |           |
| continue            | 继续执行       | c         |
| quit                | 退出           | q         |
| info all-registers  | 查看所有寄存器 | i all     |
| info registers      | 查看通用寄存器 | i r       |
| info registers t0 t1| 查看指定寄存器 | i r t0 t1 |
| info float          | 查看浮点寄存器 | i float   |

# 数据移动指令
宏指令move用来在寄存间移动数据：

	move $12, $13    # $12 = $13


采用指令集手册中的指令达到同样的目的：

	add $12, $13, 0    # $12 = $13 + 0


显然，宏指令更容易读懂。

# hi,lo寄存器

hi, lo这两个寄存器专用于乘法和除法，乘除等算数运算我们以后再讲，今天先学会如何读取和写入这两个寄存器。

读取操作：

	mfhi $t0   # move from hi to $t0
	mflo $t1   # move from lo to $t1


写入操作：

	mthi $t0    # move $t0 to hi
	mtlo $t1    # move $t1 to lo

# 编程练习

```MIPS
/* move.s: Move data between registers 
   8 --> t0 --> t1 --> t2 --> t3 --> v0 */
   .text
   .global main
   .ent main
main: li $t0, 8	# t0=8
	move $t1, $t0	# t1=t0
	move $t2, $t1	# t2=t1
	move $t3, $t2	# t3=t2
	move $v0, $t3	# v0=t3
	jr $ra		# return
	.end main
```
