---
layout: post
title:  "PMON的配置"
date:   2018-03-23 17:10:25 +0800
categories: boot
---
## PMON的配置文件
PMON的配置文件为`Targets/LS2K/conf/ls2k`。

  1. `option XXX` 设置选项;
  2. `select xxx` 选择模块和命令，PMON的命令也是以模块的形式存在的。
  
## 配置工具`pmoncfg`
配置文件`Targets/LS2K/conf/ls2k`将被交给配置工具`pmoncfg`处理。

在`zloader/Mafile.inc`中有：
```
cfg:
        ...
        cd ../Targets/${TARGET}/conf/;pmoncfg ${TARGETEL}
        ...
```
## 生成`Makefile`及头文件
`pmoncfg`对配置文件进行语法分析，然后生成`Targets/LS2K/compile/Makefile`。
根据用户的选择，将相应模块的`xxx.o`加入`OBJS`，并设置相应的依赖关系。
根据用户的选择，在`compile`目录下，还会生成一些头文件。

## 头文件目录的符号连接
`pmoncfg`还将在`Target/LS2K/compile`目录下创建符号连接:
```
target -> ../../../../Targets/LS2K/include
machine -> ../../../../sys/arch/mips/include
```
这样，源代码中的`#include`语句就可以找到相应的头文件。比如`start.S`中：
```
#include "target/ls2k.h"
```
其实相当于：
```
#include "../../../../Targets/LS2K/include/ls2k.h"
```
## 生成`ioconf.c`
`ioconf.c`是机器生成的文件，由`pmoncfg`中的`mkioconf()`函数生成。


