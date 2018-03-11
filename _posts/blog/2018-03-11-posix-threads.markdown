---
layout: post
title: "Posix Threads编程"
date: 2018-03-11 8:53:24 +0800
categories: blog
---
# 教材
*《Programming with POSIX Threads》* by David R. Butenhof, ADDISON-WESLEY, 1997

*《POSIX多线程程序设计》* 于磊 曾刚 译，中国电力出版社，2003

# Chapter 1
```c
pthread_create
```
创建一个线程，第3个参数为线程函数
```c
pthread_detach
```
指示Pthreads当线程终结时回收线程资源
```c
pthread_exit
```
终结调用线程
```c
pthread_self
```
返回调用线程的标识，一般用法：
```c
status = pthread_detach (pthread_self());
```

