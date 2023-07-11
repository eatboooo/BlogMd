---
title: Java线程
date: 2019/11/1
categories:
- 小知识
- Java
- 并发
- 线程
tags:
- Java
- 并发
- 线程
src: //eatboooo.gitee.io/img/background/Java_thread_h.jpg
---
# 线程基础
Java 并发中线程的基础知识整理。  
Java 多线程中的三大特性：原子性，可见性，顺序性。

## 1 Java 线程生命周期中有哪些状态？各状态之间如何切换？
![java_thread_status](https://gitee.com/eatboooo/BlogMd/raw/master/img/java_thread_status.png)  
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

## 2 创建线程有哪些方式？这些方法各自利弊是什么？
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

## 3 什么是 Callable 和 Future？什么是 FutureTask？
### 什么是 Callable 和 Future？
Java 5 在 concurrency 包中引入了 Callable 接口，它和 Runnable 接口很相似，但它可以返回一个对象或者抛出一个异常。

Callable 接口使用泛型去定义它的返回类型。Executors 类提供了一些有用的方法去在线程池中执行 Callable 内的任务。由于 Callable 任务是并行的，我们必须等待它返回的结果。Future 对象为我们解决了这个问题。在线程池提交 Callable 任务后返回了一个 Future 对象，使用它我们可以知道 Callable 任务的状态和得到 Callable 返回的执行结果。Future 提供了 get() 方法让我们可以等待 Callable 结束并获取它的执行结果。
### 什么是 FutureTask？
FutureTask 是 Future 的一个基础实现，我们可以将它同 Executors 使用处理异步任务。通常我们不需要使用 FutureTask 类，单当我们打算重写 Future 接口的一些方法并保持原来基础的实现是，它就变得非常有用。我们可以仅仅继承于它并重写我们需要的方法。阅读 Java FutureTask 例子，学习如何使用它。

## 4 start() 和 run() 有什么区别？可以直接调用 Thread 类的 run() 方法么？
+ run() 方法是线程的执行体。
+ start() 方法负责启动线程，然后 JVM 会让这个线程去执行 run() 方法。
### 可以直接调用 Thread 类的 run() 方法么？
+ 可以。但是如果直接调用 Thread 的 run() 方法，它的行为就会和普通的方法一样。
+ 为了在新的线程中执行我们的代码，必须使用 start() 方法。

## 5 sleep()、yield()、join() 方法有什么区别？为什么 sleep() 和 yield() 方法是静态（static）的？
### yield()
+ yield()方法和sleep()方法类似，也不会释放“锁标志”，区别在于，它没有参数，即yield()方法只是使当前线程重新回到可执行状态，所以执行yield()的线程有可能在进入到可执行状态后马上又被执行，另外yield()方法只能使同优先级或者高优先级的线程得到执行机会，这也和sleep()方法不同。
+ yield() 方法可以让当前正在执行的线程暂停，但它不会阻塞该线程，它只是将该线程从 Running 状态转入 Runnable 状态。
+ 当某个线程调用了 yield() 方法暂停之后，只有优先级与当前线程相同，或者优先级比当前线程更高的处于就绪状态的线程才会获得执行的机会。
### sleep()
+ sleep() 方法需要指定等待的时间，它可以让当前正在执行的线程在指定的时间内暂停执行，进入 Blocked 状态。
+ 该方法既可以让其他同优先级或者高优先级的线程得到执行的机会，也可以让低优先级的线程得到执行机会。
+ 但是，sleep() 方法不会释放“锁标志”，也就是说如果有 synchronized 同步块，其他线程仍然不能访问共享数据。
### join()
+ join() 方法会使当前线程转入 Blocked 状态，等待调用 join() 方法的线程结束后才能继续执行。
+ 也就是说 join() 方法会使当前线程等待调用 join() 方法的线程结束后才能继续执行。
### 为什么 sleep() 和 yield() 方法是静态（static）的？
+ Thread 类的 sleep() 和 yield() 方法将处理 Running 状态的线程。所以在其他处于非 Running 状态的线程上执行这两个方法是没有意义的。这就是为什么这些方法是静态的。它们可以在当前正在执行的线程中工作，并避免程序员错误的认为可以在其他非运行线程调用这些方法。(在 run() 方法中直接使用)

## 6 wait()方法。
+ wait() ， notify() 及 notifyAll() 只能在 synchronized 语句中使用，因为 wait() 会释放锁；
### notify() 和 notifyAll()。
+ notify() 默认的唤醒策略是：先进入 wait() 的线程先被唤醒。
+ notifyAll() 唤醒所有的等待线程，默认的唤醒策略是： LIFO （后进先出）。
+ notify() 可能导致死锁，notifyAll() 开销大。

## 7 说说 sleep() 方法和 wait() 方法区别和共同点?
+ 两者最主要的区别在于：sleep 方法没有释放锁，而 wait 方法释放了锁 。
+ 两者都可以暂停线程的执行。
+ Wait 通常被用于线程间交互/通信，sleep 通常被用于暂停执行。
+ wait() 方法被调用后，线程不会自动苏醒，需要别的线程调用同一个对象上的 notify() 或者 notifyAll() 方法。sleep() 方法执行完成后，线程会自动苏醒。或者可以使用wait(long timeout)超时后线程会自动苏醒。

## 8.1 为什么我们调用 start() 方法时会执行 run() 方法，为什么我们不能直接调用 run() 方法？
这是另一个非常经典的 java 多线程面试问题，而且在面试中会经常被问到。很简单，但是很多人都会答不上来！

new 一个 Thread，线程进入了新建状态;调用 start() 方法，会启动一个线程并使线程进入了就绪状态，当分配到时间片后就可以开始运行了。 start() 会执行线程的相应准备工作，然后自动执行 run() 方法的内容，这是真正的多线程工作。 而直接执行 run() 方法，会把 run 方法当成一个 main 线程下的普通方法去执行，并不会在某个线程中执行它，所以这并不是多线程工作。

总结： 调用 start 方法方可启动线程并使线程进入就绪状态，而 run 方法只是 thread 的一个普通方法调用，还是在主线程里执行。

## 8.2 start() 和 run() 有什么区别？可以直接调用 Thread 类的 run() 方法么？
+ run() 方法是线程的执行体。
+ start() 方法负责启动线程，然后 JVM 会让这个线程去执行 run() 方法。
### 可以直接调用 Thread 类的 run() 方法么？
+ 可以。但是如果直接调用 Thread 的 run() 方法，它的行为就会和普通的方法一样。
+ 为了在新的线程中执行我们的代码，必须使用 start() 方法。

## 9 Java 的线程优先级如何控制？高优先级的 Java 线程一定先执行吗？
Java并发_1.7中有相关问题
### Java 中的线程优先级如何控制
+ Java 中的线程优先级的范围是 [1,10]，一般来说，高优先级的线程在运行时会具有优先权。可以通过 thread.setPriority(Thread.MAX_PRIORITY) 的方式设置，默认优先级为 5。
### 高优先级的 Java 线程一定先执行吗
+ 即使设置了线程的优先级，也无法保证高优先级的线程一定先执行。
+ 原因：这是因为 Java 线程优先级依赖于操作系统的支持，然而，不同的操作系统支持的线程优先级并不相同，不能很好的和 Java 中线程优先级一一对应。
+ 结论：Java 线程优先级控制并不可靠。

## 10 什么是守护线程？为什么要用守护线程？如何创建守护线程？
### 什么是守护线程
+ 守护线程（Daemon Thread）是在后台执行并且不会阻止 JVM 终止的线程。
+ 与守护线程（Daemon Thread）相反的，叫用户线程（User Thread），也就是非守护线程。
### 为什么要用守护线程
+ 守护线程的优先级比较低，用于为系统中的其它对象和线程提供服务。典型的应用就是垃圾回收器。
### 如何创建守护线程
+ 使用 thread.setDaemon(true) 可以设置 thread 线程为守护线程。
+ 注意点：
    - 正在运行的用户线程无法设置为守护线程，所以 thread.setDaemon(true) 必须在 thread.start() 之前设置，否则会抛出 llegalThreadStateException 异常；
    - 一个守护线程创建的子线程依然是守护线程。
    - 不要认为所有的应用都可以分配给守护线程来进行服务，比如读写操作或者计算逻辑。

总结：
+ 用个比较通俗的比如，任何一个守护线程都是整个JVM中所有非守护线程的保姆。
+ 只要当前JVM实例中尚存在任何一个非守护线程没有结束，守护线程就全部工作；只有当最后一个非守护线程结束时，守护线程随着JVM一同结束工作。Daemon的作用是为其他线程的运行提供便利服务，守护线程最典型的应用就是 GC (垃圾回收器)，它就是一个很称职的守护者。
+ User和Daemon两者几乎没有区别，唯一的不同之处就在于虚拟机的离开：如果 User Thread已经全部退出运行了，只剩下Daemon Thread存在了，虚拟机也就退出了。 因为没有了被守护者，Daemon也就没有工作可做了，也就没有继续运行程序的必要了。

## 11 线程间是如何通信的？
当线程间是可以共享资源时，线程间通信是协调它们的重要的手段。Object 类中 wait(), notify() 和 notifyAll() 方法可以用于线程间通信关于资源的锁的状态。

## 12 为什么线程通信的方法 wait(), notify() 和 notifyAll() 被定义在 Object 类里？
+ Java 的每个对象中都有一个锁(monitor，也可以成为监视器) 并且 wait()、notify() 等方法用于等待对象的锁或者通知其他线程对象的监视器可用。在 Java 的线程中并没有可供任何对象使用的锁和同步器。这就是为什么这些方法是 Object 类的一部分，这样 Java 的每一个类都有用于线程间通信的基本方法
+ 简单的说，这三个方法的操作对象都是锁级别的。

---
此博客参考资料：  
+ [Snailclimb/JavaGuide](https://github.com/Snailclimb/JavaGuide)
+ [CyC2018/CS-Notes](https://github.com/CyC2018/CS-Notes)
+ [dunwu/javacore](https://github.com/dunwu/javacore)
+ [crossoverJie/JCSprout](https://github.com/crossoverJie/JCSprout)