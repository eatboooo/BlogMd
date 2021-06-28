---
title: Java 多线程系统学习 02
date: 2021/6/28
categories:
- 小知识
- Java
- 并发
- 线程
tags:
- Java
- 并发
- 线程
src: //gitee.com/eatboooo/BlogMd/raw/master/img/background/thread_h.png

---

# Java 多线程系统学习 02

### 线程的“打断”（interrupt）（设置中断标志位）

#### 线程打断的方法

1. interrupt（）
   - 打断某个线程（设置标志位）
2. isInterrupted（）
   - 查询是否被打断过（查询标志位）
3. static interrupted（）
   - 查询当前线程是否被打断过，并重制打断标志

- 这三个组合可以优雅的结束线程
- sleep wait join 的时候使用 interrupt 会抛 interruptedException ，并且jvm 会帮你重制标记位
- synchronized/lock.lock() 并不会被 interrupt（）干扰
  - 想干扰怎么办？使用这个锁： lock.lockInterruptibly()，才会被 interrupt（）干扰

### 线程的结束

- 如何优雅的结束一个线程
  - t.stop()  -> 已经被废掉了，因为太粗暴了，容易产生数据不一致的问题（直接释放全部锁，没有善后工作）
  - t.suspend() 暂停 t.resume() 继续 -> 已经被废掉了，因为暂停的时候不会释放锁，你忘了继续怎么办？
  - 依赖 volatile 修饰的字段 -> 特定场景下一种优雅的解决方案（不考虑中间过程 eg： while（volatile）{我是中间过程}
  - interrupt() -> 特定时间点检查 Thread.interrupted() 决定是否退出 ，相比 volatile 更优雅，因为 volatile 对 sleep wait join 没有特殊处理，而 interrupt 可以抛出异常

### 并发编程

#### 三大特性

1. 可见性（synchronize

   - 线程把数据读取到缓存，假如有循环，他不会每次都从主内存读（除非使用 volatile 修饰，被它修饰的内存被修改的时候会立刻刷新）
   - 即便不使用 volatile，某些语句可以触发刷新，比如 sout，因为源码里使用了 synchronized 加锁
   - 被 volatile 修饰的引用类型，修改里面的属性之后也不会刷新，要是想刷新直接用其修饰属性完事了
   - 三级缓存
   - 缓存行 - 按块读取 - 每行64byte 字节 - 缓存一致性机制 - 两个内存读一块数据，相互修改效率很低，因为互相之间会通知 - 和 volatile 无关
   - 1.8 jdk 新增 @Contended 注解，让数据在缓存行单独一行 （使用的时候需要在 jvm 加参数 `-XX:RestrictContended`) (只有1.8版本有作用)

2. 有序性 （volatile

   - 程序真的是按‘顺序’执行的吗？
   - 乱序是为了提高效率（寄存器速度是内存的100倍），指令1 去内存区数据 、指令2 就可以先执行了
     - 前提是前后两条语句没有依赖关系
     - 不影响单线程的最终一致性
     - as - if - serial （好像 - 是 - 序列化执行的，前提是单线程）

3. 原子性 （synchronize

   - 多线程访问同一份数据会产生竞争
   - i++ 把值从内存读到寄存器再写回去 并不是线程安全，可以通过上锁来实现原子性 / AtomicInteger（内部CAS -> 调用 native 方法 -> cpu 底层支持 cas（多核cpu会 lock 住））
   - 上锁的本质：把并发编程序列化
   - 悲观锁 synchron
     - 悲观的认为线程一定会被打断
     - 重量级用悲观（执行慢，线程多）
   - 乐观锁/无锁/自旋锁/CAS
     - 乐观的认为线程不会被打断
     - ABA问题：比较原来值的时候 原来值可能被改了好几次单最后还是A（比较的是引用值，里面内容可能变了），可以加一个 version 字段每次操作都++/或时间戳
     - CAS操作本身必须是原子性的，否则判断完后准备赋值的时候，可能被打断
     - 轻量级用乐观
   - 实战就用 synchronize ，优化已经很到位了

### 用户态和内核态

JVM 工作再用户态，早期 synchronize 是重量级锁，需要 JVM 申请 OS 一把锁（用户态调用内核态），现在 synchronize 做了优化，有些直接在用户态就可以完成

### markwork

synchronize 会改变 markwork（偏向锁状态时记录 线程id），markwork 记录了 synchronize、GC、hashcode 信息

### synchronize 锁升级过程

- 未开启偏向锁:new 普通对象 -> +synchronize -> 轻度竞争变成自旋锁 -> 重度竞争变成重量级锁
  - 偏向锁是没有竞争的，有竞争就会升级 （有竞争就会 锁撤销，然后变为自旋锁）
  - 重量级锁有等待队列，不需要消耗 CPU 资源
- 开启偏向锁：new 匿名偏向对象 -> 直接就是偏向锁状态 -> 轻度竞争自旋锁 -> 重度变成重量级锁
  - 偏向锁有可能直接变成重量级锁（等待时间过长 或者调用了 wait 方法）

### 锁重入

- synchronize 是可重入锁（方法里同一个锁 锁多次）
- 必须记录重入次数，因为要释放
- 偏向锁/自旋锁 记录在线程栈里 LR
- 重量级锁记录在一个对象上
