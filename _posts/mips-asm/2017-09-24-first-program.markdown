---
layout: post
title:  "1. 第一个汇编语言程序"
date: 2017-09-24 18:49:25 +0800
categories: mips-asm
order: 1
---
<a href="https://www.bilibili.com/video/av14809674/"><img src="{{ site.url }}/assets/images/video-icon.png" alt="龙芯汇编语言程序设计教程" /></a>

# 本节课的目标
+ 编写第一个汇编语言程序
+ 程序的编辑、汇编、运行与调试

# 为什么要学习汇编语言
+ 编写启动管理器
+ 修改Linux内核
+ 为新硬件编写驱动程序
+ 应用程序优化
+ Just For Fun
  
> 汇编语言不是用来写应用软件的！

# 第一个汇编语言程序
```MIPS Assembly
/* first.s:
  This is our first MIP64 program,
  Runing on Loongson 3A3000 */
  
	.text
	.global main
	
	.ent main	# Function main
main:	nop
	li $v0, 8	# v0: Return value
	jr $ra		# Return
	.end main	# End of main
```
## 注释的风格
+ 多行注释： /* ... */ 
+ 单行注释： #

## 命令的类型
+ 指令：《MIPS® Architecture For Programmers Volume II-A: The MIPS64® Instruction Set》
+ 汇编指令：也叫伪指令
+ 宏指令

# 让程序跑起来
+ 编辑

		$ vi first.s

+ 汇编连接

		$ gcc -O0 -g -o first first.s

+ 执行

		$ ./fisrt
	
+ 查看结果

		$ echo $?
	

