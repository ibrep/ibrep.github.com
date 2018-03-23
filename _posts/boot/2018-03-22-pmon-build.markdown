---
layout: post
title:  "PMON的构建系统"
date:   2018-03-22 21:10:25 +0800
categories: boot
---
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

### 编译生成`pmon.bin`

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
`pmon.bin`被`gzip`压缩成`pmon.bin.gz`，再由`bin2c`转化为`pmon.bin.c`中的`char biosdata[]`数组。

```
zloader.o: zloader.c inflate.c  malloc.c  memop.c  pmon.bin.c initmips.c
	$(CC) -c zloader.c ${ZLOADER_OPTIONS} -DMEMSIZE=${MEMSIZE}
```
在`zloader.c`中，有一条语句 `#include "pmon.bin.c"`，因此`biosdata`最终和解压函数`run_unzip(`最终被编译成`zloader.o`目标文件。

```
${START}:
	rm -f ../Targets/${TARGET}/compile/${TARGETEL}/${START}
	gcc  -DSTARTADDR=${ROMSTARTADDR} -DOUT_FORMAT=\"${OUT_FORMAT}\" -DOUT_ARCH=mips -Umips -E -P ld.script.S  > ../Targets/${TARGET}/conf/ld.script
	make -C ../Targets/${TARGET}/compile/${TARGETEL}/
	cp ../Targets/${TARGET}/compile/${TARGETEL}/${START} .
```
注意，`${START}`就是`start.o`，其源文件为`Targets/LS2K/ls2k/start.S`。

```
ejtag_rom ejtag_rom1 ejtag_ram rom: clean ${START} zloader.o 
	gcc  -DSTARTADDR=${GZROMSTARTADDR} -D_ROM1 -DOUT_FORMAT=\"${OUT_FORMAT}\" -DOUT_ARCH=mips -Umips -E -P ld.script.S  > ld.script
	${LD} -T ld.script -e start -o gzrom ${START} zloader.o 
	${CROSS_COMPILE}objcopy -O binary gzrom gzrom.bin
```
终于，`start.o`和`zloader.o`被连接到一起，生成了`gzrom`，进而用`objcopy`转换为`gzrom.bin`。

## 地址问题
在`zloader/Makefile.inc`中，定义了PMON在内存中的位置：
```
GZROMSTARTADDR?=0xffffffff8f900000
ROMSTARTADDR?=0xffffffff8f010000
```
连接生成目标文件`pmon`时，起始地址用的是`8f010000`。连接生成目标文件`gzrom`时，起始地址用的是`8f900000`。
这两个目标文件入口点都是`start.o`，就是说`start.o`被连接了两次，一次地址是`8f010000`，另一次是`8f900000`。

根据`See Mips Run`，系统启动时，入口地址是`0xBFC0 0000`，相应的物理地址是`1FC0 0000`。
在`cache`初始化之后，`0x9FC0 0000`也会被映射到同一个位置。`gzrom.bin`被写入这个位置。
就是说，`start.o`开始执行的时候，并不知道自己在`0xBFC0 0000`，而是认为自己在`8f900000`。

## 解压缩过程---真假`initmips()`
`start.o`执行完之后，将跳到`initmips()`函数中，调用`run_unzip()`将`biosdata[]`解压缩到`0x8f01 0000`。

注意这个`initmips()`并不是真正的做系统初始化工作的`initmips()`, 真正的`initmips()`是在`Targets/LS2K/ls2k/tgt_machdep.c`中定义的，现在它还只是一堆被压缩的二进制数据，存在`biosdata[]`数组中。

假的`initmips()`是在`zloader/initmips.c`文件中定义的，而这个文件不是手工编写的，是用一个`perl`脚本`genrom`自动生成的。

在`Makefile.inc`中有：
```
ifeq ("${tgt}","rom")
gencode=./genrom
endif

...

initmips.c:  ../Targets/${TARGET}/compile/${TARGETEL}/pmon
	${gencode} $< > initmips.c
```
可见，`initmips.c`文件不是人工编写的，是用`genrom`自动生成的。

`genrom`脚本打开目标文件`pmon`，找到真正的`initmips()`入口地址，并在生成`initmips.c`时，插入一段汇编代码，
跳转到真正的`initmips()`去执行初始化工作。

下面的代码片段来自`genrom`:
```C
#!/usr/bin/perl 
my ($myedata,$myend,$initmips,$mystart);
open(F,qq(objdump -x $ARGV[0]|));
while(<F>)
{
chomp;
if(/([0-9a-f]+).+_edata/){
   $myedata=qq(0x$1);
 }

if(/([0-9a-f]+).+_end$/){
   $myend=qq(0x$1);
 }
if(/([0-9a-f]+).+initmips$/){
   $myinitmips=qq(0x$1);
 }
if(/([0-9a-f]+).+\s_start$/){
   $mystart=qq(0x$1);
 }
}

void initmips(unsigned int msize,int dmsize,int dctrl)
{
    long *edata=(void *)$myedata;
    long *end=(void *)$myend;
    int *p;
	int debug=(msize==0);
//	CPU_TLBClear();
    tgt_puts("Uncompressing Bios");
    if(!debug||dctrl&1)enable_cache();
	while(1)
	{
    if(run_unzip(biosdata,$mystart)>=0)break;
	}
    tgt_puts("OK,Booting Bios\\r\\n");
    for(p=edata;p<=end;p++)
    {
        *p=0;
    }
	memset($mystart-0x1000,0,0x1000);//$mystart-0x1000 for frame(registers),memset for pretty
#ifdef NOCACHE2
	flush_cache();
#else
	flush_cache2();
#endif
    realinitmips(debug?dmsize:msize);
}

inline __attribute__((always_inline)) void realinitmips(unsigned int msize)
{
	     asm ("li  \$29,$mystart-0x4000;\\n" \\
"		       li \$2,$myinitmips;\\n" \\
"			   move \$4,\%0;\\n" \\
"			   jalr \$2;\\n" \\
"			   nop;\\n" \\
"			  1: b 1b;nop;" \\
          :
          : "r" (msize)
          : "\$29", "\$2","\$4");

}
```
