---
title: 进程间通信
layout: page
date: 2016-04-10
---
[TOC]

## 同步与互斥

同步（synchronization）：进程同步指两个以上进程基于某个条件来协调它们的活动。一个进程的执行依赖于另一
个协作进程的消息或信号，当一个进程没有得到来自于另一个进程的消息或信号时则需等待，直到消息或信号到达才被唤醒。
互斥（mutex）：进程互斥指若干个进程要使用同一共享资源时，任何时刻最多允许一个进程去使用，其他要使用该资源的进程必须等待，直到占有资源的进程释放该资源。

同步体现进程间的协作关系，互斥体现进程间的竞争关系。