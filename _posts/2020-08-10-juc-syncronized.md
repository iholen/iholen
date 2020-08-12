---
layout: post
title:  "Java并发之syncronized"
date:   2020-08-10 19:55:50 +0800
categories: juc
author: heoller
introduction: juc
---

### syncronized原理
syncronized是基于java对象加锁的。java对象创建后jvm会给该对象维护一个管程(Monitor)对象, Monitor最终又依赖操作系统维护的Mutex(互斥量)进行一系列锁操作, 比如: Monitor.Enter(获取锁)、Monitor.Exit(释放锁)等。

syncronized会被编译成monitorenter和monitorexit两条指令。其实wait/notify也是依赖于monitor对象的，这也是它们只能在同步块或同步方法中才能被调用的原因。

### 锁升级过程
早期版本, syncronized加锁时直接使用mutex加一个重量级锁, 自jdk1.5之后, syncronized优化了加锁过程，过程如下:

* 当单线程获取对象锁时，此时只会加偏向锁
* 当多线程获取对象锁时，线程在没有获取到锁时通过自旋等待后成功获取到锁，此时升级到轻量级锁。
* 当多线程获取对象锁时，线程在没有获取到锁且自旋等待后也未获取到锁，此时升级到重量级锁。

