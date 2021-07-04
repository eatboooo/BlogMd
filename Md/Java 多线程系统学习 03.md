---
title: Java 多线程系统学习 03
date: 2021/6/29
categories:
- 小知识
- Java
- 并发
- 线程
tags:
- Java
- 并发
- 线程
src: //gitee.com/eatboooo/BlogMd/raw/master/img/background/thread_h03.png


---

# Java 多线程系统学习 03

### 基础回顾

- qq.exe - 进程 （os进行资源分配的基本单位）
- 进程的最小执行单元 - 线程 （调度执行的基本单位）
- sleep（）
- yield（） 让出一下 cpu 、让一下时间片
- join（）在 t1 调用 t2.join() ，t2 执行完 t1 才执行
- synchronize 可以修饰方法
- synchronize 不能锁定常量（String Integer Long） 需要锁对象（通过 markword）（对象要final修饰）
- synchronize 既保证了原子性 又保证了可见性
- synchronize 必须是可重入锁、否则子类重写父类 synchronize 的方法再调用 super. 直接死锁
- 程序中出现异常 锁 会被释放
- volatile 可变的，保证线程可见性、禁止指令重排序
- volatile 和 mesi 要区分开
- new 一个对象的底层原理
  1. 申请内存
  2. 半初始化
  3. 初始化、调构造
  4. 把地址值给对象

### LoggerAdder

类似于 AtomicInter，但是他内部实现是一个类似分段锁的东西

- 数字特别大，分为数组，分组加，最后加一起
- 线程数很多 数字很大 适合使用 LoggerAdder
- 分段锁也是 cas 操作

### 基于 cas 新类型的锁

#### ReentrantLock

- 可重入锁、可以设定为公平锁（去等待队列检查，看看谁先来的）（默认为非公平的）
  - 公平锁并不是完全公平，也要看队列，放锁之后我光速进入队列
- reentrantLock.lock() - reentrantLock.unlock()
- 可以使用 tryLock ，如果没有申请到直接歇菜
- ren.newCondition() 可以 new 出多种条件
  - condition 本质是 new 等待队列
  - new 两个就是 两个不同的等待队列
  - 可以叫醒指定的等待队列

#### CountDownLatch

- latch = new CountDownLatch（100）
- latch.countDown()  -  latch.await()
- latch 变成零的时候 latch.await() 阻塞完毕
- 门栓的概念，latch.await() 相当于拴住

#### CyclicBarrier

- cy = new CyclicBarrier(20,new Runnable(){....})
  - 满20人发车
  - 发车执行第二个参数里的 run 方法
- cy.await() 等待乘客
  - 乘客指的是线程，十九个线程在等着了，才发车
  - 发车是20个线程一起执行

#### Phaser

- Phaser ph = new MyPhaser（）
- 一个阶段一个阶段的控制 - 栅栏
- 所有线程抵达后执行 boolen onAdvance(int phase，int person) 
  - 两个参数：（第几阶段，现阶段几个人参加）
  - ph.bulkRegister(7) 注册7个人参加
  - 还有两个 api 不细写了 - 等待下一阶段执行（）；结束阶段执行（）；
  - 如果 return true 栅栏结束

#### ReadWriteLock

- 读写锁、共享锁、排他锁
- 有些东西 读的时候特别多、写的时候特别少，防止数据不一致，因此这个锁出现了
- rwl = new ReadWriteLock（）
- rwl.readLock().lock()  /  rwl.readLock().unlock()
  - 共享锁、读锁
  - 读的时候可以共用，写不行
- rwl.writeLock().lock() / ...
  - 排他锁
  - 谁来都等着

#### Semaphore

- 信号量、信号灯
- sp = new Semaphore(2)
  - sp.acquire() 取一下，然后构造里面的2会变成1 （相当于从sp获得锁，得大于0，否则就阻塞）
  - sp.release() 释放，构造里面的1变回了2
- 限流用的（最多允许多少个线程在运行）
  - 100辆车只有2个加油站
- 可以设定为公平锁（也是看队列，并不是真正公平）

#### Exchanger

- ex = new Exchanger<>()
- String s = ex.exchange(str) （阻塞住）（一个线程可不行）
- 用于两个线程通信，交换字符串（只能两个线程）

#### LockSupport

- LockSupport.park() 停车
  - 线程正在运行的时候可以调用，会阻塞住
- LockSupport.unpark() 接着走
  - unpack() 可以在 park() 之前调用，调用之后就停不住了

### 问题来了

1. synchronize 和 ReentrantLock 有何不同
   1. syn 不需要解锁， reen 需要自己lock() unlock()
   2. reen 可以搞等待队列（公平锁）
   3. reen 底层使用 cas 实现，syn 四种状态锁升级（无 -> 偏向锁-> 轻量级锁 cas -> 重量级锁 内核态）
2. Object.notify（） 叫醒了，不释放锁，有的时候叫醒了也没用
3. Object.wait() 会让出锁
4. 在 t1 中调用 t2.join() 会等待 t2 执行完成后 t1 才会继续执行
5. A～Z，1～26 交替打印
6. ReentrantLock 底层实现
   - 使用 AQS 实现
   - CAS
7. AQS 
   - volatile int state - 这个 0 和 1 代表加锁 - 这个 state 的含义是由实现的子类来决定
   - state 之上维护了一个队列 Node
     - 双向链表
     - Node 里面装的是线程
     - head 、 tail 指针
     - 公平锁会用到这个队列
     - 入队出队是核心 - 使用 cas 实现

