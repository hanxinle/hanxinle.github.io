---
layout: post
title: 《Linux 多线程服务端编程》阅读与实践（1）
date: 2021-04-20 15:31:29
cover: book.jpg
tags:
- 网络编程
- 多线程
---

# 阅读小结

这篇博文是概括性的，目的在于理清作者的思路，作者有一套网络程序设计的教程，推荐学习。
<!-- more -->
前5章对 Linux 多线程做了细致的论述，对以下问题重点论述，解决。

线程安全的对象生命期，需要能标记其所持资源是否被占用的程序功能，直到提出 shared_ptr/weak_ptr 这一解决方案。

线程同步精要，给出精简有效的线程同步方案。对已经经过检验的经验要熟悉，不必求面面俱到。

多线程编程模型——one loop per thread，每个线程持有一个事件管理(Reactor),每个线程可以处理多个 I/O，这个做法和个人网盘项目的做法是一致的。我需要多写一些程序，体会这种方案的益处，知其然也知其所以然。

# 实践：Libevent实现多线程 Reactor 线程池

该线程池的设计遵循one loop per thread思想，每个线程维护一个 Reactor。

该线程池的链接是：

[https://github.com/hanxinle/Xcpp/tree/main/src/libevent_learn/ch3_thread_pool](https://github.com/hanxinle/Xcpp/tree/main/src/libevent_learn/ch3_thread_pool)

更进一步的实践是一个网盘应用，当前实现的网盘原型链接是

---

# 阅读小结

这篇博文是概括性的，目的在于理清作者的思路，作者有一套网络程序设计的教程，推荐学习。
<!-- more -->
前5章对 Linux 多线程做了细致的论述，对以下问题重点论述，解决。

线程安全的对象生命期，需要能标记其所持资源是否被占用的程序功能，直到提出 shared_ptr/weak_ptr 这一解决方案。

线程同步精要，给出精简有效的线程同步方案。对已经经过检验的经验要熟悉，不必求面面俱到。

多线程编程模型——one loop per thread，每个线程持有一个事件管理(Reactor),每个线程可以处理多个 I/O，这个做法和个人网盘项目的做法是一致的。我需要多写一些程序，体会这种方案的益处，知其然也知其所以然。

# 实践：Libevent实现多线程 Reactor 线程池

该线程池的设计遵循one loop per thread思想，每个线程维护一个 Reactor。

该线程池的链接是：

[https://github.com/hanxinle/Xcpp/tree/main/src/libevent_learn/ch3_thread_pool](https://github.com/hanxinle/Xcpp/tree/main/src/libevent_learn/ch3_thread_pool)

更进一步的实践是一个网盘应用，当前实现的网盘原型链接是

[https://github.com/hanxinle/XCloud-Disk](https://github.com/hanxinle/XCloud-Disk) 。