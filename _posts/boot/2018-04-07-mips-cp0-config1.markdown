---
layout: post
title:  "龙芯Cache组织与设置"
date:   2018-04-07 12:33:25 +0800
categories: boot
---
# CP0 Config1寄存器
CPU中与Cache相关的参数都可以在CP0 Config1 (register 16, select 1)中查到，这是一个只读寄存器。
Config1中与Cache相关的字段有：

| 24:22 | 21:19 | 18:16 | 15:13 | 12:10 | 9:7 |
|    IS |    IL |    IA |    DS |    DL |  DA |

## `IS`: Icache每路的组数

| Encoding |  Meaning | 备注 |
|----------|----------|------|
|        0 |       64 |      |
|        1 |      128 |      |
|        2 |      256 |      |
|        3 |      512 |      |
|        4 |     1024 |      |
|        5 |     2048 |      |
|        6 |     4096 |      |
|        7 | Reserved |      |

## `IL`: Icache 每行的大小

| Encoding | Meaning           | 备注 |
|----------|-------------------|------|
|        0 | No Icache present |      |
|        1 | 4 bytes           |      |
|        2 | 8 bytes           |      |
|        3 | 16 bytes          |      |
|        4 | 32 bytes          |      |
|        5 | 64 bytes          |      |
|        6 | 128 bytes         |      |
|        7 | Reserved          |      |

## `IA`: Icacche组相连数

| Encoding | Meaning       | 备注 |
|----------|---------------|------|
|        0 | Direct Mapped |      |
|        1 | 2-way         |      |
|        2 | 3-way         |      |
|        3 | 4-way         |      |
|        4 | 5-way         |      |
|        5 | 6-way         |      |
|        6 | 7-way         |      |
|        7 | 8-way         |      |

## `DS`: Dcache 每路的组数
意义与Icache相同。

## `DL`: Dcache 每行的大小
意义与Icache相同

## `DA`: Dcache 组相连数
意义与Icache相同

# 龙芯1C 的Cache组织
  * GS232核实现了分离的一级数据cache（Dcache）和一级指令cache（Icache）；
  * Icache和Dcache各16KB；
  * cache line大小为32 bytes；
  * Icache和Dcache均为4路组相连；
  * 替换策略为随机法；
  * Dcache采用writeback写策略，Icache不可写；
  * Icache 和 Dcache的读写通路均为64 bits，也就是一次可读写一个DWORD。

# 启用kseg0 cache
CP0 Config (register 16, select 0)的K0(2:0) 字段用来启用kseg0的缓存。
写入`0x02`关闭cache，写入`0x03`启用cache。

```
CPU_DisableCache:
	mfc0   	t0, CP0_CONFIG, 0
	and   	t0, 0xfffffff8
	or	t0, 0x2
	mtc0   	t0, CP0_CONFIG, 0
```
```
mfc0 a0, COP_0_CONFIG		/* enable kseg0 cachability */
ori  a0, a0, 0x3           // ENABLE
mtc0 a0, COP_0_CONFIG
```

# Cache的操作指令
Cache操作指令的格式为：
```
CACHE op, offset(base)
```

龙芯1C的GS232核支持以下Cache操作：

| op      | 描述                       | 目标   |
|---------|----------------------------|--------|
| 0b00000 | Index Invalidate           | Icache |
| 0b00001 | Index Writeback Invalidate | Dcache |
| 0b00101 | Index Load Tag             | Dcache |
| 0b01000 | Index Store Tag            | Icache |
| 0b01001 | Index Store Tag            | Dcache |
| 0b10000 | Hit Invalidate             | Icache |
| 0b10001 | Hit Invalidate             | Dcache |
| 0b10101 | Hit Writeback Invalidate   | Dcache |
| 0b11100 | Fetch and Lock             | Icache |
| 0b11101 | Fetch and Lock             | Dcache |

  * `Index Invalidate`: 将index指定的cache块的状态设置为无效；
  * `Index Writeback Invalidate`: 如果index指定的cache块是有效的而且是脏的，则将块的内容写会主内存，完成后，再将块儿设置为无效。如果块儿有效但不脏，则只是设置为无效。
  * `Index Load Tag`: 将index指定的cache块儿的tag读入CP0的TagLo和TagHi寄存器。如果实现了DataLo和DataHi寄存器，则同时将数据写入这两个寄存器。龙芯的GS232并没有实现DataLo和DataHi；
  * `Index Store Tag`: 将TagLo和TagHi寄存器中的tag写入index指定的Cache块儿；
  * `Hit Invalidate`: 如果cache块儿包含指定的地址，则将cache块儿的状态设置为无效；
  * `Hit Writeback Invalidate`: 如果cache块儿包含指定的地址，而且有效并且为脏，则将内容写回主内存，写完后，再将cache块儿设置为无效；如果有效但不脏，则只是设置为无效。
  * `Fecth and Lock`：如果cache不包含指定的地址，则从内存中读出并填充cache，必要时writeback，然后将状态设置为有效并且锁定；如果cache已经包含指定的地址，则锁定它。`Index Invalidate`, `Index Writeback Invalidate`, `Hit Invalidate`, `Hit Writeback Invalidate`四条命令可以清除锁定状态。`Index Store Tag`也可以清除锁定状态。
  
