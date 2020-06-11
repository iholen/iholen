---
layout: post
title:  "Java并发之CountDownLatch原理"
date:   2020-06-11 21:47:50 +0800
categories: juc
author: heoller
introduction: juc
---

## 从源码看`CountDownLatch`原理

当存在多线程执行任务，需要等待指定数量的子线程都执行完后，再执行主线程任务场景时，可以使用`CountDownLatch`类来实现。其静态内部类`Sync`继承了`AbstractQueuedSynchronizer`，实现了`AQS`类的`tryAcquireShared`和`tryReleaseShared`方法。

#### 基本使用

```java
public class CountDownLatchTest {
    public static void main(String[] args) throws InterruptedException {
        // 定义计数器值为10的CountDownLatch对象
        CountDownLatch countDownLatch = new CountDownLatch(10);
        // 创建10个子线程执行任务，最少需要10个子线程
        for (int i = 0; i < 10; i++) {
            new Thread(() -> {
                try {
                    Thread.sleep(1000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                } finally {
                    // 任务执行完后将计数器减1
                    countDownLatch.countDown();
                }
                System.out.println(Thread.currentThread().getName() + " end...");
            }, "Thread-" + i).start();
        }
        // 在此等待所有任务完成
        countDownLatch.await();
        System.out.println("Thread-main end...");
    }
}
```

#### 源码解析

* 构造一个计数器10的`CountDownLatch`对象，设置`state`值为10

```java
// step 1
public CountDownLatch(int count) {
    if (count < 0) throw new IllegalArgumentException("count < 0");
    this.sync = new Sync(count);
}
// step 2
Sync(int count) {
    setState(count);
}
```

* `countDown`以及`await`方法分析

```java
// step 1: 调用了sync实例的releaseShared方法
public void countDown() {
    sync.releaseShared(1);
}
// step 2: 接着调用了sync实例的tryReleaseShared方法
public final boolean releaseShared(int arg) {
    // 尝试将计数器state进行减一
    if (tryReleaseShared(arg)) {
        doReleaseShared();
        return true;
    }
    return false;
}
// step3: 接着查看`tryReleaseShared`方法，该方法具体实现是在`Sync`类中
private static final class Sync extends AbstractQueuedSynchronizer {
    // 省略不相关代码...
    protected boolean tryReleaseShared(int releases) {
        // 此处的for循环是为了解决在高并发场景下，CAS操作失败时，进行重试
        for (;;) {
            int c = getState();
            // 当state已经为0时，直接返回false。
            if (c == 0)
                return false;
            int nextc = c-1;
            // 基于CAS操作，保证state变量的原子性更新
            if (compareAndSetState(c, nextc))
                // 当state更新为0时，返回true。
                return nextc == 0;
        }
    }
}
// step 4: 根据tryReleaseShared方法的分析得知：
// 只有将state成功更新为0的线程，才会执行doReleaseShared逻辑，即step 2中的if块逻辑
// 在分析这个方法之前，先看下await方法，直接看step 5
private void doReleaseShared() {
    for (;;) {
        Node h = head;
        if (h != null && h != tail) {
            int ws = h.waitStatus;
            if (ws == Node.SIGNAL) {
                if (!compareAndSetWaitStatus(h, Node.SIGNAL, 0))
                    continue;            // loop to recheck cases
                unparkSuccessor(h);
            }
            else if (ws == 0 &&
                !compareAndSetWaitStatus(h, 0, Node.PROPAGATE))
                continue;
        }
        if (h == head)
            break;
    }
}
// step 5: 判断子线程是否执行完，反之将主线程阻塞。
public void await() throws InterruptedException {
    sync.acquireSharedInterruptibly(1);
}
public final void acquireSharedInterruptibly(int arg) throws InterruptedException {
    // 线程是否已被中断
    if (Thread.interrupted())
        throw new InterruptedException();
    if (tryAcquireShared(arg) < 0)
        // 将主线程阻塞
        doAcquireSharedInterruptibly(arg);
}
protected int tryAcquireShared(int acquires) { // 改方法也是在CountDownLatch重写了
    // 这里逻辑很简单，就是判断state是否为0，为0时才返回1
    // 所以如果在执行await之前，state为已经为0时，不会将主线程阻塞。
    // 有且仅当还有等待的线程时，才会将主线程阻塞。
    return (getState() == 0) ? 1 : -1;
}
private void doAcquireSharedInterruptibly(int arg) throws InterruptedException {
    // 将主线程添加到阻塞队列中
    final Node node = addWaiter(Node.SHARED);
    boolean failed = true;
    try {
        for (;;) {
            // 获取前继节点
            final Node p = node.predecessor();
            // 该队列里中只有一个主线程节点，所以此处为true
            if (p == head) {
                // 再次检查state是否为0
                int r = tryAcquireShared(arg);
                // 条件满足表示state为0，所需数量的子线程任务已执行完
                if (r >= 0) {
                    // 将head指针指向新加的主线程阻塞节点
                    setHeadAndPropagate(node, r);
                    p.next = null; // help GC
                    failed = false;
                    return;
                }
            }
            // 如果子线程未执行完，将head节点设为可通知运行状态
            // 并在第二次for循环后，设置为阻塞状态，通过LockSupport$park方法设置
            // 注意：此处阻塞之后，会在子线程执行完后，进行唤醒，唤醒后，会从此处继续执行
            // 继续执行后，会再进入一次循环，下一次循环进来时，state即为0了
            // head指针也会指向主线程阻塞的那个节点，退出循环
            if (shouldParkAfterFailedAcquire(p, node) &&
                parkAndCheckInterrupt())
                throw new InterruptedException();
        }
    } finally {
        if (failed)
            cancelAcquire(node);
    }
}
// step 6: 再回到step 4源码处
// 此方法进入条件为state为0，即子线程执行完毕, 将唤醒主线程
private void doReleaseShared() {
    for (;;) {
        Node h = head;
        // 如果阻塞队列中存在阻塞线程节点
        if (h != null && h != tail) {
            int ws = h.waitStatus;
            // 第五步已分析到，主线程阻塞前，head的waitStatus值会被修改为Node.SIGNAL
            if (ws == Node.SIGNAL) {
                // 若设置失败，通过for循环重试
                if (!compareAndSetWaitStatus(h, Node.SIGNAL, 0))
                    continue;
                // 将主线程唤醒，主线程继续执行，通过LockSupport$unpark方法设置
                unparkSuccessor(h);
            }
            else if (ws == 0 &&
                !compareAndSetWaitStatus(h, 0, Node.PROPAGATE))
                continue;
        }
        if (h == head)
            break;
    }
}
```

以上是小弟阅读源码后总结的，思路还是理出来了的，睡觉去咯，Zz...
