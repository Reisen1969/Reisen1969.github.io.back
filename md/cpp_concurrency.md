+++
author = "rayrain"
title = "C++并发"
date = "2022-05-15"
description = "C++"
toc= true
math= true
tags = [
    "C","CPP"
]

+++

## 多线程编程

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

`thread` 启动一个新线程

创建线程 `thread`

`join` :主线程等待这个线程结束

`detach` 让线程独立运行 ,成为守护线程

访问共享数据的代码片段称之为**临界区**（critical section）

`mutex`互斥加锁

条件量

`wait`

`notify_all`

本质上是线程间 共享的全局flag

```
#include <iostream>
#include <string>
#include <thread>
#include <mutex>
#include <condition_variable>
 
std::mutex m;
std::condition_variable cv;
std::string data;
bool ready = false;
bool processed = false;
 
void worker_thread()
{
    // Wait until main() sends data
    std::unique_lock lk(m);
    cv.wait(lk, []{return ready;});
 
    // after the wait, we own the lock.
    std::cout << "Worker thread is processing data\n";
    data += " after processing";
 
    // Send data back to main()
    processed = true;
    std::cout << "Worker thread signals data processing completed\n";
 
    // Manual unlocking is done before notifying, to avoid waking up
    // the waiting thread only to block again (see notify_one for details)
    lk.unlock();
    cv.notify_one();
}
 
int main()
{
    std::thread worker(worker_thread);
 
    data = "Example data";
    // send data to the worker thread
    {
        std::lock_guard lk(m);
        ready = true;
        std::cout << "main() signals data ready for processing\n";
    }
    cv.notify_one();
 
    // wait for the worker
    {
        std::unique_lock lk(m);
        cv.wait(lk, []{return processed;});
    }
    std::cout << "Back in main(), data = " << data << '\n';
 
    worker.join();
}
```



### future

`future`对象用来存储异步任务的执行结果,获取到这个对象的时候,可能异步任务还没执行.

`wait()`来等待该异步任务结束

`get()`获取该异步任务的结果



### async

假设你需要一个长时间的运算,但并不迫切的需要这个值,可以启动一个新线程来执行这个计算,但是`thread` 并没有机制获得返回值.

可以使用`async`启动一个异步任务,而且会返回一个 `future`对象.

```c++
#include <future>
#include <iostream>
int find_the_answer_to_ltuae()
{
    return 42;
}

void do_other_stuff()
{}

int main()
{
    std::future<int> the_answer=std::async(find_the_answer_to_ltuae);
    do_other_stuff();
    std::cout<<"The answer is "<<the_answer.get()<<std::endl;
}

```

 `async`是否会启动一个新线程, 标准没有定义,由编译器决定,

如果一定要启动新线程,可以加入`launch::async`参数来说明.

`std::launch::async` 在独立线程上执行

`std::launch::deferred` 在`wait`或者`get`调用时执行 ,并不是独立线程

`std::launch::async|std::launch::deferred`  在`wait`或者`get`调用时执行 ,并独立线程执行

```c++
auto f6=std::async(std::launch::async,Y(),1.2); // 在新线程上执行
auto f7=std::async(std::launch::deferred,baz,std::ref(x)); // 在wait()或get()调用时执行 
auto f8=std::async( 
	std::launch::deferred | std::launch::async, 
	baz,std::ref(x)); // 实现选择执行方式
auto f9=std::async(baz,std::ref(x));
f7.wait(); // 调用延迟函数
```



### promise

上面的例子,异步任务的结果是通过 return 返回的

但在一些时候，我们可能不能这么做：在得到任务结果之后，可能还有一些事情需要继续处理，例如清理工作。



异步任务不再直接返回计算结果，而是增加了一个`promise`对象来存放结果。

在任务计算完成之后，将总结过设置到`promise`对象上。一旦这里调用了`set_value`，其相关联的`future`对象就会就绪

```
#include <vector>
#include <thread>
#include <future>
#include <numeric>
#include <iostream>
#include <chrono>
 
void accumulate(std::vector<int>::iterator first,
                std::vector<int>::iterator last,
                std::promise<int> accumulate_promise)
{
    int sum = std::accumulate(first, last, 0);
    accumulate_promise.set_value(sum);  // Notify future
}
 
void do_work(std::promise<void> barrier)
{
    std::this_thread::sleep_for(std::chrono::seconds(1));
    barrier.set_value();
}
 
int main()
{
    // Demonstrate using promise<int> to transmit a result between threads.
    std::vector<int> numbers = { 1, 2, 3, 4, 5, 6 };
    std::promise<int> accumulate_promise;
    std::future<int> accumulate_future = accumulate_promise.get_future();
    std::thread work_thread(accumulate, numbers.begin(), numbers.end(),
                            std::move(accumulate_promise));
 
    // future::get() will wait until the future has a valid result and retrieves it.
    // Calling wait() before get() is not needed
    //accumulate_future.wait();  // wait for result
    std::cout << "result=" << accumulate_future.get() << '\n';
    work_thread.join();  // wait for thread completion
 
    // Demonstrate using promise<void> to signal state between threads.
    std::promise<void> barrier;
    std::future<void> barrier_future = barrier.get_future();
    std::thread new_work_thread(do_work, std::move(barrier));
    barrier_future.wait();
    new_work_thread.join();
}
```

### packaged_task

`packaged_task`绑定到一个函数或者可调用对象上。当它被调用时，它就会调用其绑定的函数或者可调用对象。并且，可以通过与之相关联的`future`来获取任务的结果。调度程序只需要处理`packaged_task`，而非各个函数。

任务不会自己启动,你必须调用它.

```
#include <iostream>
#include <cmath>
#include <thread>
#include <future>
#include <functional>
 
// unique function to avoid disambiguating the std::pow overload set
int f(int x, int y) { return std::pow(x,y); }
 
void task_lambda()
{
    std::packaged_task<int(int,int)> task([](int a, int b) {
        return std::pow(a, b); 
    });
    std::future<int> result = task.get_future();
 
    task(2, 9);
 
    std::cout << "task_lambda:\t" << result.get() << '\n';
}
 
void task_bind()
{
    std::packaged_task<int()> task(std::bind(f, 2, 11));
    std::future<int> result = task.get_future();
 
    task();
 
    std::cout << "task_bind:\t" << result.get() << '\n';
}
 
void task_thread()
{
    std::packaged_task<int(int,int)> task(f);
    std::future<int> result = task.get_future();
 
    std::thread task_td(std::move(task), 2, 10);
    task_td.join();
 
    std::cout << "task_thread:\t" << result.get() << '\n';
}
 
int main()
{
    task_lambda();
    task_bind();
    task_thread();
}
```

## C++内存模型

上文中提到的知识是以互斥体为中心的。为了避免竞争条件，是保证任何时候只有一个线程可以进入临界区

那些策略都是基于锁（lock-based）的：一旦有一个线程进入临界区，其他线程只能等待。

那有没有一种策略可以让其他线程不用等待，实现更好的并发呢？

答案是肯定的，这称之为免锁（lock-free）策略。不过实现这种策略要更麻烦一些，要对C++内存模型有更深入的理解，而这也是下面所要讲解的内容。



内存模型主要包含了下面三个部分：

- **原子操作**：顾名思义，这类操作一旦执行就不会被打断，你无法看到它的中间状态，它要么是执行完成，要么没有执行。
- **操作的局部顺序**：一系列的操作不能被乱序。
- **操作的可见性**：定义了对于共享变量的操作如何对其他线程可见。

事实上，开发者编写的代码和最终运行的程序往往会存在较大的差异，而运行结果与开发者预想一致，只是一种“假象”罢了。

之所以会产生差异，原因主要来自下面三个方面：

- **编译器优化**
- **CPU out-of-order执行**
- **CPU Cache不一致性**

### Memory Reorder

“Memory Reorder”包含了编译器和处理器两种类型的乱序。

![img](https://paul-pub.oss-cn-beijing.aliyuncs.com/2019/2019-12-05-cpp-memory-model/memory-reorder.png)

```
X = 0, Y = 0;

Thread 1: 
X = 1; // ①
r1 = Y; // ②

Thread 2: 
Y = 1;
r2 = X;
```

线程1中事件发生的顺序虽然是先①后②，但是对于线程2来说，它看到结果可能却是先②后①。当然，线程1看线程2也是一样的。

甚至，**当今的所有硬件平台，没有任何一个会提供完全的顺序一致（sequentially consistent）内存模型**，因为这样做效率太低了。

不同的编译器和处理器对于Memory Reorder有不同的偏好，但它们都遵循一定的原则，那就是：**不能修改单线程的行为**（[Thou shalt not modify the behavior of a single-threaded program.](https://preshing.com/20120625/memory-ordering-at-compile-time/)）。在这个基础上，它们可以做各种类型的优化。



### sequenced-before

`sequenced-before` 单线程上的关系，这是一个非对称，可传递的成对关系

```
int i = 7; // ①
i++;       // ②
```

这里的 ① sequenced-before ② 。

### happens-before

happens-before关系是sequenced-before关系的扩展，因为它还包含了不同线程之间的关系。

如果A happens-before B，则A的内存状态将在B操作执行之前就可见，这就为线程间的数据访问提供了保证。

同样的，这是一个非对称，可传递的关系。

如果A happens-before B，B happens-before C。则可推导出A happens-before C。

### synchronizes-with

synchronizes-with描述的是一种状态传播（propagate）关系。如果A synchronizes-with B，则就是保证操作A的状态在操作B执行之前是可见的。

下文中我们将看到，原子操作的acquire-release具有synchronized-with关系。

除此之外，对于锁和互斥体的释放和获取可以达成synchronized-with关系，还有线程执行完成和join操作也能达成synchronized-with关系。

最后，借助 synchronizes-with 可以达成 happens-before 关系。



![img](https://paul-pub.oss-cn-beijing.aliyuncs.com/2019/2019-12-05-cpp-memory-model/memory_model_relation.png)



## 原子

可能是使用的互斥量实现,也可能是使用的对应硬件平台的原子指令实现.

`is_lock_free()` 可以用来查询是否是无锁的.

| 函数                                          | #_flag | #_bool | 指针类型 | 整形类型 | 说明                                   |
| :-------------------------------------------- | :----- | :----- | :------- | :------- | :------------------------------------- |
| test_and_set                                  | Y      |        |          |          | 将flag设为true并返回原先的值           |
| clear                                         | Y      |        |          |          | 将flag设为false                        |
| is_lock_free                                  |        | Y      | Y        | Y        | 检查原子变量是否免锁                   |
| load                                          |        | Y      | Y        | Y        | 返回原子变量的值                       |
| store                                         |        | Y      | Y        | Y        | 通过一个非原子变量的值设置原子变量的值 |
| exchange                                      |        | Y      | Y        | Y        | 用新的值替换，并返回原先的值           |
| compare_exchange_weak compare_exchange_strong |        | Y      | Y        | Y        | 比较和改变值                           |
| fetch_add, +=                                 |        |        | Y        | Y        | 增加值                                 |
| fetch_sub, -=                                 |        |        | Y        | Y        | 减少值                                 |
| ++, --                                        |        |        | Y        | Y        | 自增和自减                             |
| fetch_or, \|=                                 |        |        |          | Y        | 求或并赋值                             |
| fetch_and, &=                                 |        |        |          | Y        | 求与并赋值                             |
| fetch_xor, ^=                                 |        |        |          | Y        | 求异或并赋值                           |

### memory_order

原子操作中,都支持一个类型为 `std::memory_order` 的可选参数。这个参数是一个枚举类型，可能的取值如下：

```
typedef enum memory_order {
    memory_order_relaxed,
    memory_order_consume,
    memory_order_acquire,
    memory_order_release,
    memory_order_acq_rel,
    memory_order_seq_cst
} memory_order;
```

首先，并非每一种`memory_order`对于每一个原子操作都有意义。它们的使用需要有特定的配合。

我们可以根据原子操作是否读写数据分为“Read”，“Write”以及“Read-Modify-Write”（读、修改、写）三类，下面是这些操作的分类。

| Operation               | Read | Write | Read-Modify-Write |
| :---------------------- | :--- | :---- | :---------------- |
| test_and_set            |      |       | Y                 |
| clear                   |      | Y     |                   |
| is_lock_free            | Y    |       |                   |
| load                    | Y    |       |                   |
| store                   |      | Y     |                   |
| exchange                |      |       | Y                 |
| compare_exchange_strong |      |       | Y                 |
| compare_exchange_weak   |      |       | Y                 |
| fetch_add, +=           |      |       | Y                 |
| fetch_sub, -=           |      |       | Y                 |
| fetch_or, \|=           |      |       | Y                 |
| ++,–                    |      |       | Y                 |
| fetch_and, &=           |      |       | Y                 |
| fetch_xor, ^=           |      |       | Y                 |

而对于每一个分类，有意义的`memory_order`参数如下。

| Operation         | Order                                                        |
| :---------------- | :----------------------------------------------------------- |
| Read              | memory_order_relaxed memory_order_consume memory_order_acquire memory_order_seq_cst |
| Write             | memory_order_relaxed memory_order_release memory_order_seq_cst |
| Read-modify-write | memory_order_relaxed memory_order_acq_rel memory_order_seq_cst |

当多个线程中包含了多个原子操作，这些原子操作因为其`memory_order`的选择不一样，将导致运行时不同的内存模型强度。从强至弱，有三种情况：

- Sequential Consistency：顺序一致性，简称 seq-cst。
- Acquire and Release：获取和释放，简称 acq-rel。
- Relaxed：松散模型。

### seq-cst 模型

当使用原子操作而又不指定`memory_order`时将使用默认的内存顺序：`memory_order_seq_cst`，因此调用这些函数时指定或者不指定这个值效果是一样的。

这是最严格的内存模型，seq-cst 有两个保证：

- 程序指令与源码顺序一致
- 所有线程的所有操作存在一个全局的顺序

这意味着：所有关于原子操作的代码都不会被乱序，你可以列出线程交错的所有可能性，即便每次执行交错的结果会不一样。但对于任意一次来说，其执行的顺序必属于这些可能性中的一个。而且，对于某一个单次执行来说，所有线程看到的顺序是一致的。

在这种模型下，每个线程中所有操作的先后关系，其顺序对于所有线程都是可见的。因此它是所有线程的全局同步。

这种模型很容易理解，但缺点是它的性能较差。因为为了实现顺序一致需要添加很多手段来对抗编译器和CPU的优化

```
#include<bits/stdc++.h> 
std::atomic<bool> x,y;
std::atomic<int> z;

void write_x_then_y()
{
    x.store(true); // ①
    y.store(true); // ②
}

void read_y_then_x()
{
    while(!y.load()); // ③
    if(x.load()) // ④
        ++z; // ⑤
}

int main()
{
    x=false;
    y=false;
    z=0;
    std::thread a(write_x_then_y);
    std::thread b(read_y_then_x);
    a.join();
    b.join();
    assert(z.load()!=0); // ⑥
}
```

发生在线程a中的时序也将同步到线程b中。对于y的store和load操作构成了synchronized-with关系。

1  happens-before 2 happens-before 3 happens-before 4

![img](https://paul-pub.oss-cn-beijing.aliyuncs.com/2019/2019-12-05-cpp-memory-model/seq_cst.png)

### acq-rel 模型

memory_order_release对应了写操作，memory_order_acquire对应了读操作，memory_order_acq_rel对应了既读又写。

同一个原子变量上的acquire和release操作将引入synchronizes-with关系。除此之外，将不再有全局的一致顺序。

- 同一个对象上的原子操作不允许被乱序。
- release操作禁止了所有在它之前的读写操作与在它之后的写操作乱序。
- acquire操作禁止了所有在它之前的读操作与在它之后的读写操作乱序。

```
#include<bits/stdc++.h> 
std::atomic<bool> x,y;
std::atomic<int> z;

void write_x_then_y()
{
    x.store(true, std::memory_order_relaxed); // ①
    y.store(true, std::memory_order_release); // ②
}

void read_y_then_x()
{
    while(!y.load(std::memory_order_acquire)); // ③
    if(x.load(std::memory_order_relaxed))
        ++z;  // ④
}

int main()
{
    x=false;
    y=false;
    z=0;
    std::thread a(write_x_then_y);
    std::thread b(read_y_then_x);
    a.join();
    b.join();
    assert(z.load()!=0); // ⑤
}
```

![img](https://paul-pub.oss-cn-beijing.aliyuncs.com/2019/2019-12-05-cpp-memory-model/rel-acq.png)

### relaxed 模型

在进行原子操作时，指定memory_order_relaxed时将使用relaxed模型。这是最弱的内存模型。

这个模型下唯一可以保证的是：**对于特定原子变量存在全局一致的修改顺序，除此以外不再有其他保**证。这意味着，即便是同样的代码，不同的线程可能会看到不同的执行顺序。

```
#include<bits/stdc++.h> 
std::atomic<bool> x,y;
std::atomic<int> z;

void write_x_then_y()
{
    x.store(true, std::memory_order_relaxed); // ①
    y.store(true, std::memory_order_relaxed); // ②
}

void read_y_then_x()
{
    while(!y.load(std::memory_order_relaxed)); // ③
    if(x.load(std::memory_order_relaxed)) // ④
        ++z;  // ⑤
}

int main()
{
    x=false;
    y=false;
    z=0;
    std::thread a(write_x_then_y);
    std::thread b(read_y_then_x);
    a.join();
    b.join();
    assert(z.load()!=0); // ⑥
}
```

这里的assert是可能会触发的。

![img](https://paul-pub.oss-cn-beijing.aliyuncs.com/2019/2019-12-05-cpp-memory-model/relaxed.png)

- 尽管所有操作都是原子的，但是所有的事件不要求存在一个全局顺序
- 同一个线程内部有happens-before规则，但是线程之间可能会看到不同的顺序

另外需要说明的是：这里问题的发生只是**理论上的可能**。如果你将上面这个代码片段编译和运行，估计你运行100次也碰不到问题的发生。但是，这并不表示问题不存在，**它只是很难发生而已**。而这也恰恰是并发系统难以开发的原因之一：很多问题在绝大部分时候都不会出现，当在极少数时候发生的时候，又很难被理解。

relaxed模型约束太小，因此常常需要结合Fence来一起使用

### Fence

Fence这个单词的中文翻译就是“栅栏”，它就像一个屏障一样，使得其前后的代码不能穿越。

Fence有三种情况：

- full fence：指定了memory_order_seq_cst或者memory_order_acq_rel。
- acquire fence：指定了memory_order_acquire。
- release fence：指定了memory_order_release。

不同类型的Fence对于乱序的保护是不一样的。我们可以将读和写的交错分成下面四种情况：

- ① Load-Load：读接着读
- ② Load-Store：先读后写
- ③ Store-Load：先写后读
- ④ Store-Store：写接着写



full fence可以防止①②④三种情况下，但是不能防止第③种情况下。

![img](https://paul-pub.oss-cn-beijing.aliyuncs.com/2019/2019-12-05-cpp-memory-model/full_fence.png)

acquire fence阻止了所有在它之前的读操作与在它之后的读写操作乱序

![img](https://paul-pub.oss-cn-beijing.aliyuncs.com/2019/2019-12-05-cpp-memory-model/acquire_fence.png)

release fence阻止了所有在它之前的读写操作与在它之后的写操作乱序

![img](https://paul-pub.oss-cn-beijing.aliyuncs.com/2019/2019-12-05-cpp-memory-model/release_fence.png)

使用fence修改relaxed的代码

```
std::atomic<bool> x,y;
std::atomic<int> z;

void write_x_then_y()
{
    x.store(true, std::memory_order_relaxed); // ①
    std::  (std::memory_order_release);
    y.store(true, std::memory_order_relaxed); // ②
}

void read_y_then_x()
{
    while(!y.load(std::memory_order_relaxed)); // ③
    std::atomic_thread_fence(std::memory_order_acquire);
    if(x.load(std::memory_order_relaxed))
        ++z;  // ④
}
```

### mutex和Fence

之前我们介绍了互斥体`mutex`：拿到`mutex`锁的线程将拥有唯一进入临界区的资格。

除了保证互斥之外，其实`mutex`的加锁和解锁之间也起到了“栅栏”的作用。因为在栅栏里面的代码是怎么都不会被优化乱序到栅栏之外（但不保证栅栏之外的内容进入到栅栏之中）。

如下图的三种情况，第一种可能会被优化成第二种。但是第二种情况不会被优化成第三种：

![img](https://paul-pub.oss-cn-beijing.aliyuncs.com/2019/2019-12-05-cpp-memory-model/mutex-fence.png)

## GCC官方的解释:

http://gcc.gnu.org/wiki/Atomic/GCCMM/AtomicSync

#### Sequentially Consistent 顺序一致性

```
 -Thread 1-       -Thread 2-
 y = 1            if (x.load() == 2)
 x.store (2);        assert (y == 1)
```



断言一定能成功,在线程1中,y的存储 happens-before x的存储

线程2看到的也一样.



```
             a = 0
             y = 0
             b = 1
 -Thread 1-              -Thread 2-
 x = a.load()            while (y.load() != b)
 y.store (b)                ;
 while (a.load() == x)   a.store(1)
    ;
```

线程2循环,等待y的值发生变化,然后改变a

线程1正在等待a发生变化

```
 -Thread 1-       -Thread 2-                   -Thread 3-
 y.store (20);    if (x.load() == 10) {        if (y.load() == 10)
 x.store (10);      assert (y.load() == 20)      assert (x.load() == 10)
                    y.store (10)
                  }
```

两个断言都一定成功

#### Relaxed宽松顺序

这个模型消除了 "再发生之前"的限制, 减少了同步

```
-Thread 1-
y.store (20, memory_order_relaxed)
x.store (10, memory_order_relaxed)

-Thread 2-
if (x.load (memory_order_relaxed) == 10)
  {
    assert (y.load(memory_order_relaxed) == 20) /* assert A */
    y.store (10, memory_order_relaxed)
  }

-Thread 3-
if (y.load (memory_order_relaxed) == 10)
  assert (x.load(memory_order_relaxed) == 10) /* assert B */
```

两个断言可能会失败.

不存在任何的happens-before, 任何一个线程都不依赖另一个线程的顺序.

唯一存在的顺序:线程2看到的线程1中一个变量的值(如x),那么它就无法看到x的更早的值.

```
-Thread 1-
x.store (1, memory_order_relaxed)
x.store (2, memory_order_relaxed)

-Thread 2-
y = x.load (memory_order_relaxed)
z = x.load (memory_order_relaxed)
assert (y <= z)
```

断言一定成功

线程2只能看到最新值,无法看到旧值.



在一定时间内,realxed load可以看到另一个线程的realxed store,这意味着,realxed 操作需要刷新缓存



当编程者只是希望一个变量是原子的,而不需要和其他线程进行同步时,就可以使用realxed模式

#### Acquire/Release 获得释放一致性

这个模式是前两种的混合, 它和sequentially consistent 相似,

它只对依赖的变量应用happens-before关系

- 同一个对象上的原子操作不允许被乱序。
- release操作禁止了所有在它之前的读写操作与在它之后的写操作乱序。
- acquire操作禁止了所有在它之前的读操作与在它之后的读写操作乱序。

```
-Thread 1-
 y.store (20, memory_order_release);

 -Thread 2-
 x.store (10, memory_order_release);

 -Thread 3-
 assert (y.load (memory_order_acquire) == 20 && x.load (memory_order_acquire) == 0)

 -Thread 4-
 assert (y.load (memory_order_acquire) == 0 && x.load (memory_order_acquire) == 10)
```





```
 -Thread 1-
 y = 20;
 x.store (10, memory_order_release);

 -Thread 2-
 if (x.load(memory_order_acquire) == 10)
    assert (y == 20);
```

线程1中,对y的写操作,不能乱序

线程2中,对y的读操作,不能乱序

断言一定成功

#### Consume 消费

比较特殊,不推荐使用

```
 -Thread 1-
 n = 1
 m = 1
 p.store (&n, memory_order_release)

 -Thread 2-
 t = p.load (memory_order_acquire);
 assert( *t == 1 && m == 1 );

 -Thread 3-
 t = p.load (memory_order_consume);
 assert( *t == 1 && m == 1 );
```

线程2中的断言成功

因为m的存储发生在p.store之前.

线程3中的断言失败

#### 全面总结

```
 -Thread 1-       -Thread 2-                   -Thread 3-
 y.store (20);    if (x.load() == 10) {        if (y.load() == 10)
 x.store (10);      assert (y.load() == 20)      assert (x.load() == 10)
                    y.store (10)
                  }
```

如果是顺序一致性同步的话,必须在系统中刷新所有可见变量,以至于所有线程看到相同的状态.

断言成立



如果是acq rel模式的话,只需要同步涉及的两个线程,这意味着同步值不能与其他线程交换.

线程1和线程2因为 x.load 同步,所以断言成立

线程2和线程3因为 y.load同步,但是线程1和线程3没有同步,所以x的值不一定是10,断言可能会失败



如果是relaxed,所有断言都有可能失败,因为根本没有同步.

#### 混合模式

```
-Thread 1-
y.store (20, memory_order_relaxed)
x.store (10, memory_order_seq_cst)

-Thread 2-
if (x.load (memory_order_relaxed) == 10)
  {
    assert (y.load(memory_order_seq_cst) == 20) /* assert A */
    y.store (10, memory_order_relaxed)
  }

-Thread 3-
if (y.load (memory_order_acquire) == 10)
  assert (x.load(memory_order_acquire) == 10) /* assert B */
```

首先,不要这样做



| **1. 16472131** |
| --------------- |
| **2. 82526461** |
| **3. 93023508** |
| **4. 09183796** |
| **5. 46548500** |
| **6. 32016383** |
