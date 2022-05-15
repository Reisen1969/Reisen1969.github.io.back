+++
author = "rayrain"
title = "并行编程"
date = "2022-05-04"
description = "parallel programming"
toc= true
math= true
tags = [
    "多核"
]

+++

参考:[C++11中的内存模型](https://www.codedump.info/post/20191214-cxx11-memory-model-1/)





https://paul.pub/cpp-memory-model/

## 多核CPU架构

![multicore](https://www.codedump.info/media/imgs/20191214-cxx11-memory-model-1/multicore.png)



## 顺序一致性Sequential Consistency 

- 每个处理器的执行顺序和代码中的顺序（program order）一样。
- 所有处理器都只能看到一个单一的操作执行顺序。

实际上还是相当于同一时间只有一个线程在工作，这种保证导致了程序是低效的，无法充分利用上多核的优点。

## 全存储排序（Total Store Ordering, 简称TSO）

在处理核心中增加写缓存，一个写操作只要写入到本核心的写缓存中就可以返回

## 松弛型内存模型（Relaxed memory models）

在松散型的内存模型中，编译器可以在满足程序单线程执行结果的情况下进行重排序（reorder),程序的执行顺序就不见得和代码中编写的一样了

## 内存栅栏（memory barrier）

由于有了缓冲区的出现，导致一些操作不用到内存就可以返回继续执行后面的操作，为了保证某些操作必须是写入到内存之后才执行，就引入了内存栅栏（memory barrier，又称为memory fence）操作。内存栅栏指令保证了，在这条指令之前所有的内存操作的结果，都在这个指令之后的内存操作指令被执行之前，写入到内存中。也可以换另外的角度来理解内存栅栏指令的作用：显式的在程序的某些执行点上保证SC。

## 关系术语

### sequenced-before

sequenced-before用于表示单线程之间，两个操作上的先后顺序，这个顺序是非对称、可以进行传递的关系。

它不仅仅表示两个操作之间的先后顺序，还表示了操作结果之间的可见性关系。两个操作A和操作B，如果有A sequenced-before B，除了表示操作A的顺序在B之前，还表示了操作A的结果操作B可见。

### happens-before

与sequenced-before不同的是，happens-before关系表示的不同线程之间的操作先后顺序，同样的也是非对称、可传递的关系。

如果A happens-before B，则A的内存状态将在B操作执行之前就可见。在上一篇文章中，某些情况下一个写操作只是简单的写入内存就返回了，其他核心上的操作不一定能马上见到操作的结果，这样的关系是不满足happens-before的。

### synchronizes-with

synchronizes-with关系强调的是变量被修改之后的传播关系（propagate），即如果一个线程修改某变量的之后的结果能被其它线程可见，那么就是满足synchronizes-with关系的。

显然，满足synchronizes-with关系的操作一定满足happens-before关系了。

## C++11内存模型

![c++model](https://www.codedump.info/media/imgs/20191214-cxx11-memory-model-2/c++model.png)

```
enum memory_order {
    memory_order_relaxed, //宽松
    memory_order_consume,
    memory_order_acquire, //用来修饰一个读操作，表示在本线程中，所有后续的关于此变量的内存操作都必须在本条原子操作完成后执行。
    memory_order_release, //用来修饰一个写操作，表示在本线程中，所有之前的针对该变量的内存操作完成后才能执行本条原子操作。
    memory_order_acq_rel, //同时包含memory_order_acquire和memory_order_release标志
    memory_order_seq_cst  //一致性内存模型 默认
};
```

原子操作分为三类

读load: memory_order_relaxed memory_order_consume memory_order_acquire memory_order_seq_cst

写store:  memory_order_relaxed   memory_order_release   memory_order_seq_cst

读-改-写 :以上全部



### memory_order_relaxed

宽松顺序

- 针对一个变量的读写操作是原子操作；
- 不同线程之间针对该变量的访问操作先后顺序不能得到保证，即有可能乱序。

```
x = 0; y = 0;
// Thread 1:
r1 = y.load(std::memory_order_relaxed); // A
x.store(r1, std::memory_order_relaxed); // B
// Thread 2:
r2 = x.load(std::memory_order_relaxed); // C 
y.store(42, std::memory_order_relaxed); // D
```

可能会出现r1 = r2 = 42 的结果

这个内存模型的典型应用是计数器递增



### memory_order_acquire

获得操作

用来修饰一个读操作，表示在本线程中，所有后续的关于此变量的内存操作都必须在本条原子操作完成后执行

如 lock()

![read-acquire](https://www.codedump.info/media/imgs/20191214-cxx11-memory-model-2/read-acquire.png)

### memory_order_release

释放操作

用来修饰一个写操作，表示在本线程中，所有之前的针对该变量的内存操作完成后才能执行本条原子操作。

如unlock()

![write-release](https://www.codedump.info/media/imgs/20191214-cxx11-memory-model-2/write-release.png)

### memory_order_acq_rel





### memory_order_consume

消费操作

上方的acq和rel的粒度太大了

```
#include <thread>
#include <atomic>
#include <cassert>
#include <string>
 
std::atomic<std::string*> ptr;
int data;
 
void producer()
{
    std::string* p  = new std::string("Hello");
    data = 42;
    ptr.store(p, std::memory_order_release);
}
 
void consumer()
{
    std::string* p2;
    while (!(p2 = ptr.load(std::memory_order_consume)))
        ;
    assert(*p2 == "Hello"); // 绝无出错： *p2 从 ptr 携带依赖
    assert(data == 42); // 可能也可能不会出错： data 不从 ptr 携带依赖
}
 
int main()
{
    std::thread t1(producer);
    std::thread t2(consumer);
    t1.join(); t2.join();
}
```

### memory_order_seq_cst

```
// 线程 1 ：
x.store(1, std::memory_order_seq_cst); // A
y.store(1, std::memory_order_release); // B
// 线程 2 ：
r1 = y.fetch_add(1, std::memory_order_seq_cst); // C
r2 = y.load(std::memory_order_relaxed); // D
// 线程 3 ：
y.store(3, std::memory_order_seq_cst); // E
r3 = x.load(std::memory_order_seq_cst); // F
```

### 与volatile的关系

[https://www.codedump.info/post/20191214-cxx11-memory-model-1/]: 





### C++线程库



并发:只存在一个处理器

并行:存在多个处理器

并行是并发的子集,统称为并发

**进程**（英语：process），是指计算机中已运行的程序。进程为曾经是分时系统的基本运作单位。在面向进程设计的系统（如早期的UNIX，Linux 2.4及更早的版本）中，进程是程序的基本执行实体；

**线程**（英语：thread）是操作系统能够进行运算调度的最小单位。它被包含在进程之中，是进程中的实际运作单位。

在默认的情况下，我们写的代码都是在进程的主线程中运行，除非开发者在程序中创建了新的线程。

当我们只有一个处理器时，所有的进程或线程会分时占用这个处理器。但如果系统中存在多个处理器时，则就可能有多个任务并行的运行在不同的处理器上。

任务会在何时占有处理器，通常是由操作系统的调度策略决定的





多线程API  由操作系统提供.

C++ 11 中,增加了多线程的支持



创建线程 thread

join :主线程等待这个线程结束

detach 让线程独立运行 ,成为守护线程



访问共享数据的代码片段称之为**临界区**（critical section）

### 互斥锁

mutex 互斥量 加锁 , 加锁和解锁是有代价的 ,锁的粒度尽量要小

我们用锁的**粒度**（granularity）来描述锁的范围。**细粒度**（fine-grained）是指锁保护较小的范围，**粗粒度**（coarse-grained）是指锁保护较大的范围。出于性能的考虑，我们应该保证锁的粒度尽可能的细。并且，不应该在获取锁的范围内执行耗时的操作，例如执行IO。如果是耗时的运算，也应该尽可能的移到锁的外面。



### 条件变量

wait

notify_all

本质上是线程间 共享的全局flag



### 异步

使用async 而不是 thread 启动一个任务

它会返回一个`future`对象。`future`用来存储异步任务的执行结果

需要注意的是，默认情况下，`async`是启动一个新的线程，还是以同步的方式（不启动新的线程）运行任务，这一点标准是没有指定的，由具体的编译器决定。如果希望一定要以新的线程来异步执行任务，可以通过`launch::async`来明确说明。



future 来存储一个异步任务的执行结果, 哪怕这个异步任务还没开始执行,

就可以定义一个 future来接受 这个异步任务的结果



需要进行一次长时间的运算,但是你现在不急的要,可以启动一个新线程来执行这个运算,

但是 thread 并没有机制获取返回值, 这里就需要async 来实现了.

C++ 中的算法可以加上一个 新参数,并行执行算法













