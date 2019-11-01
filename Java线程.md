---
title: Java线程
categories:
- 小知识
- Java
- 并发
- 线程
tags:
- Java
- 并发
- 线程
src: //111.231.250.175/img/background/PHP_login_h.png
---
# 1线程基础
Java并发中线程的基础知识整理
## 1.1Java 线程生命周期中有哪些状态？各状态之间如何切换？
![java_thread_status](http://111.231.250.175/img/thumb/java_thread_status.png)  
java.lang.Thread.State 中定义了 6 种不同的线程状态，在给定的一个时刻，线程只能处于其中的一个状态。  
- 开始（New） - 还没有调用 start() 方法的线程处于此状态。
- 可运行（Runnable） - 已经调用了 start() 方法的线程状态。此状态意味着，线程已经准备好了，一旦被线程调度器分配了 CPU 时间片，就可以运行线程。
- 阻塞（Blocked） - 阻塞状态。线程阻塞的线程状态等待监视器锁定。处于阻塞状态的线程正在等待监视器锁定，以便在调用 Object.wait() 之后输入同步块/方法或重新输入同步块/方法。
- 等待（Waiting） - 等待状态。一个线程处于等待状态，是由于执行了 3 个方法中的任意方法：
    - Object.wait()
    - Thread.join()
    - LockSupport.park()
- 定时等待（Timed waiting） - 等待指定时间的状态。一个线程处于定时等待状态，是由于执行了以下方法中的任意方法：
    - Thread.sleep(sleeptime)
    - Object.wait(timeout)
    - Thread.join(timeout)
    - LockSupport.parkNanos(timeout)
    - LockSupport.parkUntil(timeout)
- 终止(Terminated) - 线程 run() 方法执行结束，或者因异常退出了 run() 方法，则该线程结束生命周期。死亡的线程不可再次复生。

##  1.2创建线程有哪些方式？这些方法各自利弊是什么？
创建线程主要有三种方式：
1. 继承 Thread 类
    - 定义 Thread 类的子类，并重写该类的 run() 方法，该 run() 方法的方法体就代表了线程要完成的任务。因此把 run() 方法称为执行体。
    - 创建 Thread 子类的实例，即创建了线程对象。
调用线程对象的 start() 方法来启动该线程。  
1. 实现 Runnable 接口
    - 定义 Runnable 接口的实现类，并重写该接口的 run() 方法，该 run() 方法的方法体同样是该线程的线程执行体。
    - 创建 Runnable 实现类的实例，并以此实例作为 Thread 对象，该 Thread 对象才是真正的线程对象。
    - 调用线程对象的 start() 方法来启动该线程。
1. 通过 Callable 接口和 Future 接口
    - 创建 Callable 接口的实现类，并实现 call() 方法，该 call() 方法将作为线程执行体，并且有返回值。
    - 创建 Callable 实现类的实例，使用 FutureTask 类来包装 Callable 对象，该 FutureTask 对象封装了该 Callable 对象的 call() 方法的返回值。
    - 使用 FutureTask 对象作为 Thread 对象的 target 创建并启动新线程。
    - 调用 FutureTask 对象的 get() 方法来获得子线程执行结束后的返回值
三种创建线程方式对比
    - 实现 Runnable 接口优于继承 Thread 类，因为根据开放封闭原则——实现接口更便于扩展；
    - 实现 Runnable 接口的线程没有返回值；而使用 Callable / Future 方式可以让线程有返回值。
## 1.3什么是 Callable 和 Future？什么是 FutureTask？
#### 什么是 Callable 和 Future？
Java 5 在 concurrency 包中引入了 Callable 接口，它和 Runnable 接口很相似，但它可以返回一个对象或者抛出一个异常。

Callable 接口使用泛型去定义它的返回类型。Executors 类提供了一些有用的方法去在线程池中执行 Callable 内的任务。由于 Callable 任务是并行的，我们必须等待它返回的结果。Future 对象为我们解决了这个问题。在线程池提交 Callable 任务后返回了一个 Future 对象，使用它我们可以知道 Callable 任务的状态和得到 Callable 返回的执行结果。Future 提供了 get() 方法让我们可以等待 Callable 结束并获取它的执行结果。
#### 什么是 FutureTask？
FutureTask 是 Future 的一个基础实现，我们可以将它同 Executors 使用处理异步任务。通常我们不需要使用 FutureTask 类，单当我们打算重写 Future 接口的一些方法并保持原来基础的实现是，它就变得非常有用。我们可以仅仅继承于它并重写我们需要的方法。阅读 Java FutureTask 例子，学习如何使用它。