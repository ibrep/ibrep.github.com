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
pthread_create()
```
创建一个线程，第3个参数为线程函数。
```c
pthread_detach()
```
指示Pthreads当线程终结时回收线程资源。
```c
pthread_exit()
```
终结调用线程。
```c
pthread_self()
```
返回调用线程的标识，一般用法：
```c
status = pthread_detach (pthread_self());
```

# Chapter 2
```c
pthread_join()
```
阻塞调用者，直到指定的线程终止。

# Chapter 3
```c
PTHREAD_MUTEX_INITIALIZER
```
初始化mutex的宏，用于静态初始化。用这个宏静态初始化的mutex，不需要销毁。
```c
pthread_mutex_init()
pthread_mutex_destroy()
```
用于动态初始化和销毁mutex。

```c
int pthread_mutex_lock()
int pthread_mutex_trylock()
int pthread_mutex_unlock()
```
mutex的使用方法很简单，用`pthread_mutex_lock()` 或者 `pthread_mutex_trylock()`
加锁，处理共享数据，然后调用`pthread_mutex_unlock()`解锁。

如果mutex已经被别的线程锁定了，那么`pthread_mutex_lock()`讲导致调用线程阻塞。

如果mutex已被锁定，那么`pthread_mutex_trylock()`将返回错误状态`(EBUSY)`，而不是
阻塞。

## 条件变量
```C
pthread_cond_t cond = PTHREAD_COND_INITIALIZER;
int pthread_cond_init()
int pthread_cond_destroy()
```
用于创建和销毁条件变量。

```C
int pthread_cond_wait()
int pthread_cond_timedwait()
```
等待条件变量。每个条件变量都必须与一个特定的mutex相关联。当一个线程等待条件变量
时，mutex必须是锁定的。等待操作在阻塞线程之前，会解锁mutex，返回之前会重新锁定
mutex。

# Chapter 4
三种主要的线程编程模型：
  1. Pipeline
  2. Work crew
  3. Client/Server

# Chapter 5
```C
pthread_once_t once_control = PTHREAD_ONCE_INIT;
int pthread_once (pthread_once_t *once_control, void (*init_routine) (void));
```
完成做一次而且只做一次的任务。

