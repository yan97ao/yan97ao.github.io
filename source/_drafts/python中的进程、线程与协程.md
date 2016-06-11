---
title: python中的进程、线程与协程
date: 2016-06-05 20:44:52
tags:
	- python
	- 并发
	- 底层
categories: tech
---


首先回忆一下操作系统上学到的，进程是怎么来的？
对，是fork来的，fork之后返回两次，在父进程中返回子进程pid，在子进程中返回0。
此时父子进程的地址空间还是一致的，后续随着子进程对内存空间的改写，触发COW，走上不同的轨迹。
通过上面的回忆，基本可以记起来，进程是拥有独立地址空间的调度单元。


<!-- more -->

[1] https://zhuanlan.zhihu.com/p/20167077
[2] http://python.jobbole.com/85050/
[3] https://segmentfault.com/a/1190000001813992
[4] 《python核心编程》3e 第四章
