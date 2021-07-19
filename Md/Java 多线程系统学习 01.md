---
title: Java 多线程系统学习 01
date: 2021/6/24
categories:
- 小知识
- Java
- 并发
- 线程
tags:
- Java
- 并发
- 线程
src: //tva1.sinaimg.cn/large/0074GEKogy1gs7jp0mw14j30zk0k04qp.jpg
---
# Java 多线程系统学习 01

### 线程历史 - 对 cpu 性能压榨历史

- 单进程人工切换
  - 纸袋机
- 多进程批处理
  - 多个任务批量执行 （a、b、c、d 依次执行 a阻塞后面就白给）
- 多进程并行处理
  - 程序写在不同的内存位置上来回切换
- 多线程 
  - 一个程序内部不同任务的来回切换
  - slector - epoll
- 纤程/协程
  - 绿色线程、用户管理的线程（不是os管理的）

#### 小问题

- 什么是进程
  - os进行资源分配的基本单位
- 什么是线程
  - 调度执行的基本单位
  - 多个线程共享同一个进程里的资源
  - 线程切换
    - T1 -> T2
      - T1 放入 cache
      - T2 执行
      - T2 放入 cache
      - T1 执行
  - 单核 CPU 设定多线程是否有意义
    - 有，程序 sleep 的时候 可以开第二个线程压榨 cpu
  - 线程数是不是设置的越大越好
    - 不是，上下文切换也是消耗资源的
  - 线程池中线程数量设置多少最合适
    - 根据 cpu 计算能力 （32核 开 32个线程？ 也不一定，要看机器具体使用情况）
    - 出于安全角度，cpu 利用率也不用 100%，防止意外发生
    - N threads = N cpu * U cpu *（1+W/C）
      - W 等待时间、C计算时间、U cpu 期望CPU利用率、N cpu 处理器核数、N threads 设置线程数
      - W / C 使用工具来测试 或者 日志记录
- 什么是纤程/协程
- 什么是程序
  - qq.exe -> 双击 -> os 找到可执行文件 -> load 到内存 (一个进程)  -> 需要执行的时候 -> 找到我们的主线程扔给 cpu

### 在 Java 如何创建线程

1. 写一个类继承 Thread，实现 run 方法，new 我们写的类.start（）

2. 写一个类实现 Runnable 接口，重写 run 方法，new Thread（new 我们写的类）.start()

   - 两种方式那个更合适，答案是实现接口的，因为更灵活

3. 使用 Lambda 表达式

   ```java
   new Thread(()->{
   	System.out.println("Im run!")
   });
   ```

4. 线程池（详情看后续）

   ```java
   ExecutorService service = Executors.newCachedThreadPool();
   service.execute(()->{
   	System.out.println("This is my method");
   });
   service.shutdown();
   ```

5. 写一个类实现Callable<T>接口（实现 call（）方法）（这种方法可以拿到返回值）

   ```java
   // 介绍最简单的用法
   ExecutorService service = Executors.newCachedThreadPool();
   Future<T> f= service.submit(new 刚刚写的类);
   T t=f.get(); // 阻塞住！线程执行完后才能拿到值
   service.shutdown();
   // 第二种
   FutureTask<T> f = new FutureTask(new 刚刚写的类);
   Thread t = new Thread(f);
   t.start();
   T t = task.get(); // 也是阻塞住！
   ```

其实这几种方式本质上都是 new Thread 对象，然后调用他的 start（）方法

### 线程的状态

#### 六种状态

1. new（刚刚创建，还没有启动
2. runnable（可运行状态，由线程调度器可以安排执行
3. wating（等待被唤醒  eg：park（）-（unpark(t2) 可以唤醒 t2 线程）
4. time wating（隔一段时间后自动唤醒  eg：sleep（）
5. blocked（被阻塞，正在等待锁  eg: synchronized
6. terminated（线程结束  eg：join（）等待线程结束

lock.lock()  让线程进入 wating 状态，synchronized 让线程进入 blocked 状态

