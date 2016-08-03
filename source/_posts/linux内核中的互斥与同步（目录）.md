---
title: linux内核中的互斥与同步
dte: 2016-06-19 12:37:57
tags:
	- kernel
	- concurrency
categories: tech
---

产生并发的原因：
* 中断
* 软中断和tasklet
* 内核抢占
* 睡眠和用户空间同步
* SMP

内核的互斥和同步手段：
* 原子操作
* 信号量
* RCU机制
* 内存和优化屏障
* 读写锁
* 互斥量
* 等待队列
* 完成量


参考：
[1] 深入Linux内核架构
[2] 深入Linux设备驱动程序内核机制
[3] Linux内核设计与实现 3e
[4] RTFSC
