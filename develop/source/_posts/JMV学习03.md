---
title: JVM 03
date: 2021/7/20
categories:
- 小知识
- Java
- JVM
tags:
- Java
- JVM
src: //eatboooo.gitee.io/img/background/common_01_h.png
---

# JVM 03

小知识

```java
int i = 8;
// bipish 8 - 把 8 压栈
// isotre_1 - 把栈顶的数出栈，出栈之后放到局部变量表下标为1的位置中，此时下标1对应的是i，所以就是把i复值为8
i = i++;
// iload_1 - 局部变量表下标为1的位置拿出来压栈，8又进来了
// iinc 1 by 1 - 局部变量表下标为1的位置数加一
// isotre_1 - 同上面的 isotre_1，弹栈赋值
// return
// 最终 i 为 8
```



```java
int i = 8;
// bipish 8 - 把 8 压栈
// isotre_1 - 把栈顶的数出栈，出栈之后放到局部变量表下标为1的位置中，此时下标1对应的是i，所以就是把i复值为8
i = ++i;
// iinc 1 by 1 - 局部变量表下标为1的位置数加一
// iload_1 - 局部变量表下标为1的位置拿出来压栈，8又进来了
// isotre_1 - 同上面的 isotre_1，弹栈赋值
// return
// 最终 i 为 9
```



# Runtime Data Area and Instruction Set

jvms 2.4 2.5

## 指令集分类

1. 基于寄存器的指令集
2. 基于栈的指令集
   Hotspot中的Local Variable Table = JVM中的寄存器

## Runtime Data Area

PC 程序计数器

> 存放指令位置
>
> 虚拟机的运行，类似于这样的循环：
>
> while( not end ) {
>
> ​	取PC中的位置，找到对应位置的指令；
>
> ​	执行该指令；
>
> ​	PC ++;
>
> }

JVM Stack

1. Frame - 每个方法对应一个栈帧

   1. Local Variable Table 局部变量表
      1. 参数
      2. 方法内变量
      3. 只要不是 static 方法，就有 this
   2. Operand Stack 操作数栈
      对于long的处理（store and load），多数虚拟机的实现都是原子的
      jls 17.7，没必要加volatile
   3. Dynamic Linking 动态连接
      https://blog.csdn.net/qq_41813060/article/details/88379473 
      jvms 2.6.3
      指到运行时常量池符号链接
   4. return address
      a() -> b()，方法a调用了方法b, b方法的返回值放在什么地方

Heap

Method Area

1. Perm Space (<1.8)
   字符串常量位于PermSpace
   FGC不会清理
   大小启动的时候指定，不能变
2. Meta Space (>=1.8)
   字符串常量位于堆
   会触发FGC清理
   不设定的话，最大就是物理内存

Runtime Constant Pool

Native Method Stack

Direct Memory

> JVM可以直接访问的内核空间的内存 (OS 管理的内存)
>
> NIO ， 提高效率，实现zero copy

思考：

> 如何证明1.7字符串常量位于Perm，而1.8位于Heap？
>
> 提示：结合GC， 一直创建字符串常量，观察堆，和Metaspace



## 常用指令

store

load

pop

mul

sub

invoke

1. InvokeStatic

   - 执行静态方法

2. InvokeVirtual

   - 调用普通方法
   - 自带多肽

3. InvokeInterface

   - 多肽调用，如下

     ```java
     List list = new ArrayList();
     list.add(a);
     ```

4. InovkeSpecial
   可以直接定位，不需要多态的方法
   private 方法 ， 构造方法

5. InvokeDynamic
   JVM最难的指令
   lambda表达式或者反射或者其他动态语言scala kotlin，或者CGLib ASM，动态产生的class，会用到的指令

   - java 支持动态语言，运行时可能会产生很多 class

