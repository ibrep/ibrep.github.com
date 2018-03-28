---
layout: post
title:  "PMON/kernel的常见的宏定义"
date:   2018-03-23 21:10:25 +0800
categories: boot
---
## PMON
```
 #define NSREG(x) x     /* in pmon/dev/ns16550.h */
```

## kernel

```
#define CKSEG1ADDR(a)  (CPHYSADDR(a) | CKSEG1)   /* 把一个地址转换为 0xffffffffb....... */
#define CKSEG1 _CONST64_(0xffffffffa0000000)
#define CPHYSADDR(a) ((_ACAST32_(a)) & 0x1fffffff)
```
