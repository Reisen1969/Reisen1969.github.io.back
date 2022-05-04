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
