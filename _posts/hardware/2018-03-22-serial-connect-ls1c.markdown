---
layout: post
title:  "串口连接智龙开发板V2.1"
date:   2018-03-22 01:58:25 +0800
categories: hardware
---
# 在Loongnix下通过串口连接智龙开发板
## 接线方式
USB串口调试线接开发板串口（在SPI flash下方），白、绿、黑三根线从左到右排列，红线悬空。

## 软件设置
用screen做串口终端比较方便，需要做如下设置：
```C
[brep@Loongson ~]$ sudo gpasswd -a brep dialout  # 将用户brep加入dialout组
[brep@Loongson ~]$ sudo chmod o+rw /dev/ttyUSB0  # 更改/dev/ttyUSB0访问权限
[brep@Loongson ~]$ screen /dev/ttyUSB0 115200
```
要退出screen，可以按 `Ctrl-A \`。
靠近网口的按钮是复位按钮，按下复位按钮应该能看到通过串口传来的输出信息。

## 更改`/dev/ttyUSB0`的默认权限
`/dev/ttyUSB0`的默认权限是`rw-rw----`，每次都要更改权限才能用普通用户访问，太麻烦了。可以通过创建一条定制`udev`规则解决这个问题：
```
[brep@Loongson ~]$ sudo vi /etc/udev/rules.d/50-udev-default.rules
[brep@Loongson ~]$ cat /etc/udev/rules.d/50-udev-default.rules 
# serial
KERNEL=="tty[A-Z]*|pppox*|ircomm*|noz*", GROUP="uucp", MODE="0666"
```

