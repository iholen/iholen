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

* monitor在Java虚拟机(HotSpot)中的实现
```c
ObjectMonitor() {
  _header = NULL;
  _count = 0; // 记录线程获取锁的次数
  _waiters = 0,
  _recursions = 0;
  _object = NULL;
  _owner = NULL; // 指向持有ObjectMonitor对象的线程
  _WaitSet = NULL; // java对象调用wait方法时，处于wait状态的线程，会被加入到_WaitSet
  _WaitSetLock = 0 ;
  _Responsible = NULL ;
  _succ = NULL ;
  _cxq = NULL ;
  FreeNext = NULL ;
  _EntryList = NULL ; // 处于等待锁block状态的线程，会被加入到该列表
  _SpinFreq = 0 ;
  _SpinClock = 0 ;
  OwnerIsThread = 0 ;
}
```

### 锁升级过程
早期版本, syncronized加锁时直接使用mutex加一个重量级锁, 自jdk1.5之后, syncronized优化了加锁过程，过程如下:

* 当单线程获取对象锁时，此时只会加偏向锁
* 当多线程获取对象锁时，线程在没有获取到锁时通过自旋等待后成功获取到锁，此时升级到轻量级锁。
* 当多线程获取对象锁时，线程在没有获取到锁且自旋等待后也未获取到锁，此时升级到重量级锁。

以下是Mark Word在32位虚拟机上的不同状态：<br>
![Mark Word](/iholen/assets/images/mark-word.png)

我们再通过示例程序来看看变化过程

> 引入jol包

```xml
<dependency>
  <groupId>org.openjdk.jol</groupId>
  <artifactId>jol-core</artifactId>
  <version>0.12</version>
</dependency>
```

* 无锁-001

```java
Object obj = new Object();
// 00000001 00000000 00000000 00000000 (小端模式输出)
// 00000000 00000000 00000000 00000001 (大端模式输出)
// 此处对象的hashCode没有打印出来是因为打印方式是懒打印，由C语言实现的
System.out.println(ClassLayout.parseInstance(obj).toPrintable());
```

* 偏向锁-101

由于JVM启动时，也会存在大量的同步块以及多个线程竞争场景，为了减少锁升级过程带来的开销， 会延迟启用偏向锁。
> JDK 1.6之后，默认开启偏向锁
>> 开启偏向锁: -XX:+UseBiasedLocking -XX:BiasedLockingStartupDelay=0<br>
>> 关闭偏向锁: -XX:-UseBiasedLocking

```java
// VM options: -XX:BiasedLockingStartupDelay=0
Object obj = new Object();
// 无锁状态-处于匿名偏向状态-101
// 00000101 00000000 00000000 00000000
System.out.println(ClassLayout.parseInstance(obj).toPrintable());
// 偏向锁-101
synchronized (obj) {
    // 00000101 01110000 10000000 01111100
    System.out.println(ClassLayout.parseInstance(obj).toPrintable());
}
```

> tips:
>> 获取到偏向锁之后，调用对象的hashCode方法时，此时会升级为轻量级锁，因为偏向锁状态时，获取不到对象的hashCode，需要升级到轻量级锁后从栈空间的LockRecord中获取。

* 轻量级锁-00

```java
Object obj = new Object();

new Thread(() -> {
    synchronized (obj) {
        // 偏向锁 00000101 10010000 00001010 11011100
        System.out.println(ClassLayout.parseInstance(obj).toPrintable());
    }
}).start();

try {
    Thread.sleep(2000);
} catch (InterruptedException e) {
    e.printStackTrace();
}

new Thread(() -> {
    synchronized (obj) {
        // 轻量级锁 01001000 11111000 00110011 00001010
        System.out.println(ClassLayout.parseInstance(obj).toPrintable());
    }
}).start();
```

* 重量级锁-10

```java
Object obj = new Object();

new Thread(() -> {
    synchronized (obj) {
        try {
            Thread.sleep(2000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        // 重量级锁 00001010 00101100 10000010 11111101
        System.out.println(ClassLayout.parseInstance(obj).toPrintable());
    }
}).start();

new Thread(() -> {
    synchronized (obj) {
        try {
            Thread.sleep(2000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        // 重量级锁 00001010 00101100 10000010 11111101
        System.out.println(ClassLayout.parseInstance(obj).toPrintable());
    }
}).start();
```