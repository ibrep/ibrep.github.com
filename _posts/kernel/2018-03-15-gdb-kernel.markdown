---
layout: post
title:  "用GDB调试内核"
date:   2018-03-15 13:15:25 +0800
categories: kernel
---
# 使用GDB跟踪调试内核
```
qemu -kernel /boot/vmlinuz -initrd /boot/ramfs.img -s -S
```
  * 选项 `-s: shorthand for -gdb tcp::1234`，如果不想使用`1234`端口可以用 `-gdb tcp:xxxx` 来取代 `-s` 选项；
  * 选项 `-S: freeze CPU at startup (use 'c' to start execution)`

# 在另外一个`shell`窗口
```
$> gdb
 (gdb) file linux-4.14/vmlinux  # 在gdb界面中target remote 之前加载内核符号表
 (gdb) target remote:1234       # 建立gdb和gdbserver之间的连接，按c让qemu上的Linux继续运行
 (gdb) break start_kernel       # 断点的设置可以在target remote之前，也可以在之后
```
