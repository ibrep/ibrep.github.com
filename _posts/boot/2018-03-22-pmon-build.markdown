---
layout: post
title:  "PMON的构建系统"
date:   2018-03-22 21:10:25 +0800
categories: boot
---
# PMON的构建系统
## 构建 PMON
### 编译PMON之前先要编译`tools/pmconfig`:
```
apt-get install bison
apt-get install flex
apt-get install xtuils-dev    # for makedepend
cd tools/pmoncfg
make
cp pmoncfg /usr/local/bin
```
### 编译 PMON
```
cd zloader.ls2k/
make cfg all tgt=rom CROSS_COMPILE=mipsel-linux-
```
最后生成 gzrom.bin，将gzrom.bin文件烧写进flash芯片。

## 构建过程分析
### 追踪Makefile
`zloader.ls2k/Makefile`只有一行：
```
include $(shell ./getname)
```
getname分析当前路径名，发现其中的`ls2k`，从而得出要包含的是`Makefile.ls2k`。

`Makefile.ls2k`内容为：
```
TARGET=LS2K
TARGETEL=ls2k
export START=start.o
MEMSIZE=128
ZLOADER_OPTIONS=-mips3

include Makefile.inc
```
定义了几个重要的符号，比如`TARGET`和`START`，然后包含`Makefile.inc`，这才是要用的`Makefile`。

### 构建过程分析

在`Makefile.inc`中，定义了PMON在内存中的位置：
```
GZROMSTARTADDR?=0xffffffff8f900000
ROMSTARTADDR?=0xffffffff8f010000
```
最重要的构建目标是`cfg`：
```
cfg:
	# DO NOT DELETE
	perl -i -ne 'print;exit if(/^# DO NOT DELETE/);' ../lib/libc/Makefile 
	perl -i -ne 'print;exit if(/^# DO NOT DELETE/);' ../lib/libm/Makefile 
	perl -i -ne 'print;exit if(/^# DO NOT DELETE/);' ../lib/libz/Makefile 
	mkdir -p ../Targets/${TARGET}/compile
	cd ../Targets/${TARGET}/conf/;pmoncfg ${TARGETEL}
	make -C ../Targets/${TARGET}/compile/${TARGETEL}/ depend clean
```
构建cfg时，创建了 `Target/LS2K/compile/ls2k`目录，在此目录下执行pmoncfg时，生成了真正的`Makefile`，
而它又包含了源码顶层目录的`Makefile.inc`, 然后执行`make`命令，生成了真正的目标文件`pmon`。

```
OBJS= ...

CFILES= ...

all: pmon

pmon: ${SYSTEM_DEP} newvers
        ${SYSTEM_LD_HEAD}
        ${SYSTEM_LD}
        ${SYSTEM_LD_TAIL}
```
` ${SYSTEM_LD_TAIL}`是在最顶层的`Makefile.inc`中定义的，其中有：
```
SYSTEM_LD_TAIL= @${SIZE} $@; chmod 755 $@ ; \
                ${OBJCOPY} -O binary $@ $@.bin
```
此时，`pmon`被转换位`pmon.bin`。

### 打包压缩过程分析
回到 `zloader.ls2k/Makefile.inc`，继续分析压缩打包过程。

```
pmon.bin.c:  ../Targets/${TARGET}/compile/${TARGETEL}/pmon.bin
	gzip $< -c > pmon.bin.gz
	./bin2c pmon.bin.gz pmon.bin.c biosdata
```
`pmon.bin`被压缩成`pmon.bin.gz`，再由`bin2c`转化为`pmon.bin.c`中的`biosdata[]`数组。

```
ifeq ("${tgt}","rom")
gencode=./genrom
endif

...

initmips.c:  ../Targets/${TARGET}/compile/${TARGETEL}/pmon
	${gencode} $< > initmips.c
```
`initmips.c`文件不是人工编写的，是自动用`genrom`生成的，

