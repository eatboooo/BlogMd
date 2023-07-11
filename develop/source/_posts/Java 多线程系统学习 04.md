---
title: Java 多线程系统学习 04
date: 2021/7/5
categories:
- 小知识
- Java
- 并发
- 线程
tags:
- Java
- 并发
- 线程
src: //eatboooo.gitee.io/img/background/common_01_h.png
---
<meta name="referrer" content="no-referrer"/>

# Java 多线程系统学习 04

### AQS

- 最核心的是 voaltile int state，具体代表什么要看子类是如何实现的，像 ReentrantLock 实现代表的是是否加锁
- VarHandle
  - 普通属性可以进行原子操作
  - 比反射快，直接操作二进制码

### ThreadLocal

- `ThreadLocal<Person> tl = new ThreadLocal()`
- 线程独有的一个值
- 底层实现：set 的时候，设到了当前线程的 map 中了
- 用途：用于声明式事物，保证同一个 connection （多线程怎么办？）
- 不用的值必须要 tl.remove（），因为 map 里的值会一直都在

### 强引用

- 平时声明的变量就是强引用
- 垃圾回收的时候会调用 finalize（）

### 软引用

- 当一个对象只有软引用指向他，只有系统内存不够用的时候才会回收他（可以设置 jvm 堆内存来测试）
- 可以用作缓存

### 弱引用

- 只要遭遇到 GC 就会被回收（二话不说）
- 一般应用在容器里
- WeakHashMap
- ![ThreadLocal弱引用应用](http://ww1.sinaimg.cn/large/0074GEKogy1gs6fmppmmaj61ky0v0kjm02.jpg)

### 虚引用

- 管理堆外内存的
  - 堆外内存是不被 jvm 管理的
  - 监测 queue 做到自动回收堆外内存
  - JDK Unsafe类可以直接分配内存，但是必须手动回收
- 一般给写 jvm 的人用的
- GC 看到就干掉是肯定的
- new PhantomReference（new Preson（），new Queue（））
  - 一个虚引用指向对象（但是 get 不到，太虚了）
  - 还有一个队列，当虚引用被回收的时候该队列就会有值

### 为高并发准备的容器

小演变：

- Vector -> ArrayList -> CurrentLinkedQueue...
- HashTabel -> HashMap -> Collections.synchronizedMap -> CurrentHashMap

#### Vector

- jdk1.0 类似于现在的 list
- 自带锁，基本不用

#### HashTable

- jdk1.0 类似于现在的 map
- 自带锁，基本不用

#### HashMap

- 不带锁

#### Collections.synchronizedMap

- 解决 HashMap 不带锁的问题

#### CurrentHashMap

- 写的效率没有 Collections.synchronizedMap 和 HashTable 高
- 读的效率非常猛
- 内部 cas 实现

#### ArrayList

- 不加锁的 Vector

#### CurrentLinkedQueue

- 给多线程准备的
- poll（），如果没有值则返回 null
  - 原子性的，内部使用 cas 实现
- 可以用于多线程卖票

#### CurrentSikpListMap

- 由于 tree 使用 cas 实现太复杂，所以有了跳表实现的容器
- 弥补 CurrentHashMap 无序的不足

#### CopyOnWriteList

- 写的时候加锁，拷贝 **原来的数组** 到 **新的数组**（新数组长度+1）
- 读的时候不加锁
- 应用场景：读特别多，写特别少的场景（有点类似读写锁）

#### BlockingQueue

- 阻塞队列
- CurrentLinkedQueue
  - offer() 往里添加，和  add（）的区别是 add 加不进去的时候（有界的Queue）会抛异常，offer 会返回一个 boolean，不会报异常
  - peek（）取，不会 remove
  - poll（）取，remove
  - 参考上面写的
- LinkedBlockingQueue
  - put()  如果满了会阻塞住（linked 不会满，int 类型最大值的个数）
  - take() 如果空了会阻塞住
- ArrayBlockingQueue
  - 有界的
  - 对应无界的 linked
- DelayQueue
  - 按照等待时间进行排序
  - 你的任务需要实现 Delay 接口 （里面有时间啥的）
  - 一般用于按时间进行任务调度
  - 本质上是 PriorityQueue
- PriorityQueue
  - 内部实现了排序，堆排序，小根堆
- SynchronousQueue
  - 容量为零（add直接抛异常）
  - 和 Exchanger 很像
  - put（） 值 不 take（）（拿出来）的话会一直阻塞
- TransferQueue
  - 传递一些内容，有容量的
  - 厉害的地方是有一个 transfer（东西）
    - 装完东西，等待其他线程取走，我才做自己的事情

### Queue 和 List 的区别

- Queue 添加了上述这些对线程友好的 Api
- offer、peek、pull
- put、take（阻塞）

