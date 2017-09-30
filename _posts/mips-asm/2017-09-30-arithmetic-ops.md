---
layout: post
title:  "3. 算术运算"
date: 2017-09-25 00:49:25 +0800
categories: mips-asm
order: 3
---
<a href="https://www.bilibili.com/video/av14809674/" target="_blank"><img src="{{ site.url }}/assets/images/video-icon.png" alt="龙芯汇编语言程序设计教程" /></a>

# 本节目标
+ 了解数据类型
+ 掌握算术运算指令

# 数据类型

| 类型 | 位数 | 字节数 | 记号 |
|------|------|--------|------|
| byte | 8    | 1      | b    |
| halfword | 16 | 2 | h |
| word | 32 | 4 | w |
| doubleword | 64 | 8 | d |

# 整型算术指令

## 加法
```MIPS
    add $t2, $t1, $t0     # t2 = t1 + t0
    addi $t2, $t1, 8      # t2 = t1 + 8
    addu $t2, $t1, $t0    # unsigned, t2 = t1 + t0
    addiu $t2, $t1, 8     # unsigned, t2 = t1 + 8
```

doubleword(64 bit):

	dadd, daddi, daddu, daddiu


## 减法
```MIPS
sub $t2, $t1, $t0    # t2 = t1 - t0
subu $t2, $t1, $t0   # unsigned, t2 = t1 - t0
```

doubleword(64 bit):

	dsub, dsubu

## 乘法
```MIPS
mul $t2, $t1, $t0  # t2 = t1 * t0
mult $t2, $t1      # t2 * t1 ---> HI,LO
multu $t2, $t1     # unsigned, t2 * t1 ---> HI,LO
```

doubleword(64bit):

	dmult, dmultu

## 除法
```MIPS
div $t2, $t1        # t2/t1 --> LO, t2%t1 ---> HI
divu $t2, $t1       # unsigned, # t2/t1 --> LO, t2%t1 ---> HI
```

doubleword(64bit):

	ddiv, ddivu

## 有算 \sqrt{x}, \sqrt[n]{x},log(x), a^x, sin(x), cos(x) 的指令吗？
答案是没有！

所有其它运算都要转换成加减乘除等算术运算，比如：

	sin(x) = x - \frac{x^3}{3!} + \frac{x^5}{5!} - \frac{x^7}{7!} + \frac{x^9}{9!} - \dots

# 编程练习

	(a^2 - b^2)/2 = ?

```MIPS
/* calc.s */
	.text
	.global main
	.ent main
main:	li $t0, 6	   # a --> t0
	li $t1, 4	   # b --> t1
	mul $t2, $t0, $t0  # t2 = a^2
	mul $t3, $t1, $t1  # t3 = b^2
	sub $a0, $t2, $t3  # a0 = t2 - t3 = a^2 - b^2
	li $a1, 2	   # a1 = 2
	div $a0, $a1	   # (a^2 - b^2) / 2 --> lo
                           # remainder --> hi
	mflo $v0
	jr $ra
	.end main
```

例如：

<img src="{{ site.url width="320" }}/assets/images/mips-asm/lesson2-formula-2.png" />

# 更多的GDB命令：

| 命令 | 作用 | 简记 |
|------|------|------|
| list | 显示源码 | l |
| disassemble | 反汇编 | disas |
| step | 执行一行源码 | s |
| stepi | 执行一条指令 |  |

# 除数为0的问题
被零除属于严重错误，会发生核心转储：

	$ ./calc
	Floating point exception (core dumped)

查看core dump情况：
```
$ coredumpctl list calc
TIME                            PID   UID   GID SIG PRESENT EXE
Thu 2017-09-28 18:59:29 CST    1328  1000  1000   8 * /home/brep/Lectures/lesson3/calc
```

用gdb调试core dump文件：

	$ coredumpctl gdb
