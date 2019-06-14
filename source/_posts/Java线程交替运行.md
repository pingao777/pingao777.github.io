---
title: Java线程交替运行
date: 2019-06-05 10:13:15
categories: 技术人生
tags: [Java, 多线程]
---
> 有这样一个面试题：多线程打印AB字符

这玩意但凡有点Java基础的人，都会想到这是考察线程`wait`和`notify`，那么具体怎么做呢？如果长时间不写多线程程序，冷不丁的来一下，还真写不出。

先来复习下`wait`、`notify`的概念：

> wait: Causes the current thread to wait until either another thread invokes the notify() method or the notifyAll() method for this object, or a specified amount of time has elapsed.
> The current thread must own this object's monitor.

> notify: Wakes up a single thread that is waiting on this object's monitor. If any threads are waiting on this object, one of them is chosen to be awakened. The choice is arbitrary and occurs at the discretion of the implementation. A thread waits on an object's monitor by calling one of the wait methods.This method should only be called by a thread that is the owner of this object's monitor.

> notifyAll: Wakes up all threads that are waiting on this object's monitor.

由注释可以得知，wait将会让出锁，进入等待状态，直到其他线程调用notify或notifyAll。需要指出的是，这两个方法都必须在获取锁的状态下调用。

wait的使用范式如下：

```java
synchronized (obj) {
   while (<condition does not hold>)
       obj.wait(timeout);
   ... // Perform action appropriate to condition
}
```

回到最初的问题，可以启动两个线程，为他们分配一个名字`name`，分别为A和B，设置一个变量`ticket`保存着下一个可运行的线程名，只有`name == ticket`的线程才有权运行，这样只要改变`ticket`的值就可以控制线程的运行了，具体代码如下：

```java
public class Main {
    public static void main(String[] args) {
        new PrintChar('A').start();
        new PrintChar('B').start();
    }

    private static class PrintChar extends Thread {
        private static final Object lock = new Object();
        private static char running = 'A';
        private char name;

        public PrintChar(char name) {
            this.name = name;
        }

        @Override
        public void run() {
            for (int i = 0; i<10;i++) {
                synchronized (lock) {
                    while (name != running) {
                        try {
                            // System.out.print(" <" + name + " waiting> ");
                            lock.wait();
                        } catch (InterruptedException e) {
                            e.printStackTrace();
                        }
                    }
                    System.out.print(name);
                    loop();
                    lock.notify();
                }
            }
        }

        private void loop() {
            if (running == 'B') {
                running = 'A';
            } else {
                running += 1;
            }
        }
    }
}
```

为了观察线程的运行等待状态，我们将注释放开，得到下面的结果，

```
A <A waiting> B <B waiting> A <A waiting> B <B waiting> A <A waiting> B ...
```

作为一个问题的延伸，考虑下面的问题：

> 多线程打印ABCDE

小伙伴们，你想到了吗？
