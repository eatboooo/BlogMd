---
title: JVM 01
date: 2021/7/12
categories:
- 小知识
- Java
- JVM
tags:
- Java
- JVM
src: //eatboooo.gitee.io/img/background/common_01_h.png
---

# JVM 01

### java从编码到执行

- javac -》java 文件编译成 class 文件
- class 被 classLoader 装载到内存 （java 类库也会）
- 调用字节码解释器 或 JIT 及时编译器
- 由执行引擎开始执行
- 之后就是os硬件

### 其他

- jvm 与 java 无关
- 任何语言编译成 class，jvm 都可以执行
- jvm 只跟 calss 有关系
- jvm 是一种规范
- jdk 包含 jre 包含 jvm

### 常见 jvm 实现

- Hotspot
  - oracle
- Jrockit
  - 曾经号称最快 jvm
  - 现在被 oracle 收购，合并于 oracle
- TaobaoVM
  - Hotspot 深度定制版本
- LiquidVM
  - 直接针对硬件
- azul zing
  - 垃圾回收的业界标杆
  - 被 Hotspot 参考，才有了 ZGC

### Class File Format

### Class Loading

1. 加载过程

   1. Loading

      ```
      一个class被load到内存后，产生两块内容，1 二进制到内存 2 生成class对象指向二进制内存
      class 对象在 metaspace 中
      ```

      1. 双亲委派，主要出于安全来考虑

      2. LazyLoading 五种情况

         1. –new getstatic putstatic invokestatic指令，访问final变量除外

            –java.lang.reflect对类进行反射调用时

            –初始化子类的时候，父类首先初始化

            –虚拟机启动时，被执行的主类必须初始化

            –动态语言支持java.lang.invoke.MethodHandle解析的结果为REF_getstatic REF_putstatic REF_invokestatic的方法句柄时，该类必须初始化

      3. ClassLoader的源码

         1. findInCache -> parent.loadClass -> findClass()

      4. 自定义类加载器

         1. extends ClassLoader
         2. overwrite findClass() -> defineClass(byte[] -> Class clazz)
         3. 加密
         4. <font color=red>第一节课遗留问题：parent是如何指定的，打破双亲委派，学生问题桌面图片</font>
            1. 用super(parent)指定
            2. 双亲委派的打破
               1. 如何打破：重写loadClass（）
               2. 何时打破过？
                  1. JDK1.2之前，自定义ClassLoader都必须重写loadClass()
                  2. ThreadContextClassLoader可以实现基础类调用实现类代码，通过thread.setContextClassLoader指定
                  3. 热启动，热部署
                     1. osgi tomcat 都有自己的模块指定classloader（可以加载同一类库的不同版本）

      5. 混合执行 编译执行 解释执行

         1. 检测热点代码：-XX:CompileThreshold = 10000

   2. Linking 

      1. Verification
         1. 验证文件是否符合JVM规定
      2. Preparation
         1. 静态成员变量赋默认值
      3. Resolution
         1. 将类、方法、属性等符号引用解析为直接引用
            常量池中的各种符号引用解析为指针、偏移量等内存地址的直接引用

   3. Initializing

      1. 调用类初始化代码 <clinit>，给静态成员变量赋初始值，执行静态语句块

2. 小总结：

   1. load - 默认值 - 初始值
   2. new - 申请内存 - 默认值 - 初始值

## JMM（java 内存模型）

![存储器层次结构](http://cdn.cdnjson.com/tva4.sinaimg.cn/large/0074GEKogy1gsi295qk4hj312e0ks4cv.jpg)

### 硬件层数据一致性

协议很多

intel 用MESI

https://www.cnblogs.com/z00377750/p/9180644.html

现代CPU的数据一致性实现 = 缓存锁(MESI ...) + 总线锁

读取缓存以cache line为基本单位，目前64bytes

位于同一缓存行的两个不同数据，被两个不同CPU锁定，产生互相影响的伪共享问题

伪共享问题：JUC/c_028_FalseSharing

使用缓存行的对齐能够提高效率

### 乱序问题

CPU为了提高指令执行效率，会在一条指令执行过程中（比如去内存读数据（慢100倍）），去同时执行另一条指令，前提是，两条指令没有依赖关系

https://www.cnblogs.com/liushaodong/p/4777308.html

写操作也可以进行合并

https://www.cnblogs.com/liushaodong/p/4777308.html

JUC/029_WriteCombining

乱序执行的证明：JVM/jmm/Disorder.java

原始参考：https://preshing.com/20120515/memory-reordering-caught-in-the-act/

### 如何保证特定情况下不乱序

硬件内存屏障 X86

>  sfence:  store| 在sfence指令前的写操作当必须在sfence指令后的写操作前完成。
>  lfence：load | 在lfence指令前的读操作当必须在lfence指令后的读操作前完成。
>  mfence：modify/mix | 在mfence指令前的读写操作当必须在mfence指令后的读写操作前完成。

> 原子指令，如x86上的”lock …” 指令是一个Full Barrier，执行时会锁住内存子系统来确保执行顺序，甚至跨多个CPU。Software Locks通常使用了内存屏障或原子指令来实现变量可见性和保持程序顺序

JVM级别如何规范（JSR133）

> LoadLoad屏障：
> 	对于这样的语句Load1; LoadLoad; Load2， 
>
> 	在Load2及后续读取操作要读取的数据被访问前，保证Load1要读取的数据被读取完毕。
>
> StoreStore屏障：
>
> 	对于这样的语句Store1; StoreStore; Store2，
> 								
> 	在Store2及后续写入操作执行前，保证Store1的写入操作对其它处理器可见。
>
> LoadStore屏障：
>
> 	对于这样的语句Load1; LoadStore; Store2，
> 								
> 	在Store2及后续写入操作被刷出前，保证Load1要读取的数据被读取完毕。
>
> StoreLoad屏障：
> 	对于这样的语句Store1; StoreLoad; Load2，
>
> ​	 在Load2及后续所有读取操作执行前，保证Store1的写入对所有处理器可见。

volatile的实现细节

1. 字节码层面
   ACC_VOLATILE

2. JVM层面
   volatile内存区的读写 都加屏障

   > StoreStoreBarrier
   >
   > volatile 写操作
   >
   > StoreLoadBarrier

   > LoadLoadBarrier
   >
   > volatile 读操作
   >
   > LoadStoreBarrier

3. OS和硬件层面
   https://blog.csdn.net/qq_26222859/article/details/52235930
   hsdis - HotSpot Dis Assembler
   windows lock 指令实现 | MESI实现

synchronized实现细节

1. 字节码层面
   ACC_SYNCHRONIZED
   monitorenter monitorexit
   monitorexit 有两个，因为有异常的时候也要释放锁
   
2. JVM层面
   C C++ 调用了操作系统提供的同步机制

3. OS和硬件层面
   X86 : lock cmpxchg / xxx
   [https](https://blog.csdn.net/21aspnet/article/details/88571740)[://blog.csdn.net/21aspnet/article/details/](https://blog.csdn.net/21aspnet/article/details/88571740)[88571740](https://blog.csdn.net/21aspnet/article/details/88571740)

