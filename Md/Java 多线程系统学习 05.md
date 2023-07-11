---
title: Java 多线程系统学习 05
date: 2021/7/7
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

# Java 多线程系统学习 05

### CompletableFeature

``` java
a = CompletableFeature.supplyAsync(()-myMethod1());
b = CompletableFeature.supplyAsync(()-myMethod2());
c = CompletableFeature.supplyAsync(()-myMethod3());
CompletableFeature.allOf(a,b,c).join()
// 还有 thenapply 给后续操作
```

- 将多个任务组合运行，管理多个 Feature 结果

### 线程池

#### ThreadPoolExecutor

- 普通线程
- 维护两个集合
  - 线程的集合
  - 任务的集合
- 七个参数
  - corePoolSize，核心线程数
  - maximumPoolSize，最大线程数
  - keepAliveTime，生存时间
  - TimeUnit.SECONDS，时间单位
  - Queue，任务队列
  - Factory，线程工厂
  - ，拒绝策略（一般自己写，下面是 jdk 提供的四种）
    - 抛异常
    - 不抛异常，静悄悄执行
    - 排队时间最久的扔掉（最老的没有意义，实时游戏）
    - 让调用他的线程去执行

#### SingleThreadExceptor

- 只有一个线程
- 保证任务顺序
- 为什么要有这个？
  - 任务队列
  - new 线程 有生命周期的管理，使用这个比较方便

#### CachedThreadPool

- 一般不用（除非确保不会线程堆积）
- 阿里不用
- 核心线程为 0，最大线程数为 integer.max

#### FixedThreadPool

- 阿里不用（让自己推算需要多少线程）
- 线程数是固定的

#### ScheduledThreadPool

- 定时任务线程池
- 内部使用 DelayQueue
- 几个参数
  - 第一个任务执行之前隔多久
  - 执行间隔时间
  - 时间单位

#### ForkJoinPool

- 分解汇总任务

#### WorkStealingPool

- 每个线程有自己单独的任务队列
- 自己任务执行完了会去其他线程拿 （poll（）方法 加锁的）
- 实则是 ForkJoinPool ，不过他帮设置了一些参数

#### ParallelStream

- 并行流处理

### JMH

- 量化性能

### Disruptor

- 分类、瓦解
- 单机最快MQ
- 单线程每秒600w订单