---
title: IO
layout: page
date: 2015-09-23
---
[TOC]

## UNIX IO 模型

1. 阻塞I/O
2. 非阻塞I/O
3. I/O复用（select、poll、linux 2.6种改进的epoll）
4. 信号驱动IO（SIGIO）
5. 异步I/O（POSIX的aio_系列函数）

![unix io 模型](http://7xjtfr.com1.z0.glb.clouddn.com/unix_io_model.jpg)

POSIX把I/O操作划分成两类：

1. 同步I/O: 同步I/O操作导致请求进程阻塞，直至操作完成
2. 异步I/O: 异步I/O操作不导致请求阻塞

所以Unix的前四种I/O模型都是同步I/O, 只有最后一种才是异步I/O。