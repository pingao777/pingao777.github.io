---
title: Java“锁”记
date: 2019-03-07 14:12:29
categories: 三言两语
tags: [并发, 分布式, Java]
---
## 内置锁和显示锁

内置锁其实是相对显示锁来说的，说白了内置锁就是synchronized所代表Java原生锁机制，Jdk5.0之后又引入了`Lock`及其子类`ReentrantLock`这样一种新的锁机制。从加锁和内存语义上二者一样，只不过后者添加了一些其他功能，可以实现诸如轮询锁、超时锁和中断锁的功能。
```java
public interface Lock {
    void lock();
    void lockInterruptibly() throw InterruptedException;
    boolean tryLock();
    boolean tryLock(long timeout, TimeUnit unit)
        throw InterruptedException;
    void unlock();
    Condition newCondition();
}
```
如果内置锁是一个`Lock`的话，它只有lock()和unlock()方法。从锁的基本属性上说，内置锁和显示锁都是可重入的，内置锁是非公平的，显示锁还可以设置为公平的。

tryLock和lock的区别是前者获得锁返回true，获取不到返回false，都是立马返回，而后者如果获取不到将会阻塞到那里。

从ReentrantLock衍生出来一个ReentrantReadWriteLock，为啥要有读写锁呢？其实是基于这样的原则，读写和写写是会引起线程安全问题的，所以都需要同步，前者是因为可见性，后者是因为一致性，但是读读是不需要同步的，所以讲读写拆分开来以提高性能。这就好比原来大家都排一个队，现在拆成两个队，自然排队等待的时间就短了。

## 闭锁
闭锁就像一个门，等待一个“事件”开门（结束状态），在开门之前不允许任何人（线程）通过，在此之前大家只能在城门前面等待。只不过城门可以开也可以闭，闭锁只是一次性的。

具体到Java中，闭锁的实现就是`CountDownLatch`，它可以用来实现等待某种条件满足后才把线程放行的功能，比如资源就绪、服务启动、某个操作执行等等。

## 信号量

## 栅栏

## 原子变量
