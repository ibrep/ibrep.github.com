
---
layout: post
title:  "内核装载地址的确定"
date:   2018-03-16 22:58:25 +0800
categories: kernel
---
# 压缩内核和未压缩内核的装载地址
  1. 压缩内核vmlinuz的装载地址由符号`VMLINUZ_LOAD_ADDRESS`给出；
  2. 未压缩内核vmlinux的装载地址由符号`VMLINUX_LOAD_ADDRESS`给出。

# 1. 未压缩内核装载地址`VMLINUX_LOAD_ADDRESS`的定义
在`arch/mips/loongson64/Platform`中，有

```
load-$(CONFIG_LOONGSON_MACH3X) += 0xffffffff80200000
```

而在 `.config`中，有：

```
CONFIG_LOONGSON_MACH3X=y
```

从而有：`load-y=0xffffffff80200000`

在`arch/mips/Makefile`中有：

```
KBUILD_CPPFLAGS += -DVMLINUX_LOAD_ADDRESS=$(load-y)
bootvars-y = VMLINUX_LOAD_ADDRESS=$(load-y) \
             VMLINUX_ENTRY_ADDRESS=$(entry-y) \
             PLATFORM="$(platform-y)" \
             ITS_INPUTS="$(its-y)"
```

因此，`VMLINUX_LOAD_ADDRESS=0xffffffff80200000`

# 2. 压缩内核装载地址`VMLINUZ_LOAD_ADDRESS`的确定
`arch/mips/boot/compressed/calc_vmlinuz_load_addr.c` 被编译为 `calc_vmlinuz_load_addr`命令，由这个这个命令计算出压缩内核的装载地址。

这个命令在`arch/mips/boot/compressed/Makefile`中被调用：

```
VMLINUZ_LOAD_ADDRESS = $(shell $(obj)/calc_vmlinuz_load_addr \
           $(obj)/vmlinux.bin $(VMLINUX_LOAD_ADDRESS))
```

在 `arch/mips/boot/compressed/calc_vmlinuz_load_addr.c`中，是这样进行计算的：

```C
vmlinuz_load_addr = vmlinux_load_addr + vmlinux_size;
/* 
 * Align with 64KB: KEXEC assume kernel align to PAGE_SIZE
 */
vmlinuz_load_addr += (65536 - vmlinux_size % 65536);
```

