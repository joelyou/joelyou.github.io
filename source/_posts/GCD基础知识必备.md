---
title: GCD基础知识必备
date: 2018-04-26 10:47:39
tags: GCD
---

# GCD初识
谈到iOS多线程，一般都会谈到四种方式：pthread、NSThread、GCD和NSOperation。其中，苹果推荐也是我们最经常使用的无疑是GCD。所以掌握gcd的用法也是每个ios开发者必备的技能

GCD是一套低层级的C API，通过 GCD，开发者只需要向队列中添加一段代码块(block或C函数指针)，而不需要直接和线程打交道。GCD在后端管理着一个线程池，它不仅决定着你的代码块将在哪个线程被执行，还根据可用的系统资源对这些线程进行管理。
特点：

1. 快，更快的内存效率，因为线程栈不暂存于应用内存
2. 稳，提供了自动的和全面的线程池管理机制，稳定而便捷。
3. 准，提供了直接并且简单的调用接口，使用方便，准确。

## 队列和任务
学习gcd，肯定会接触到各种词汇，队列和任务

### 队列
调度队列是一个对象，它会以first-in、first-out的方式管理您提交的任务。GCD有三种队列类型：

1. 串行队列，让任务一个接着一个地执行（一个任务执行完后，再执行下一个任务），GCD中获得串行有2种途径(使用dispatch_queue_create函数，并指定队列类型DISPATCH_QUEUE_SERIAL,主队列)，放在主队列中的任务，都会放到主线程中执行
2. 并行队列，可以让多个任务并发（同时）执行（自动开启多个线程同时执行任务）,并发队列会基于系统负载来合适地选择并发执行这些任务；并发功能只有在异步（dispatch_async）函数下才有效，因为异步函数才具备开启新线程的能力，而同步函数只能在当前线程中执行不具备开启线程的能力；并发队列获取2种途径 （手动创建并发队列，dispatch_queue_create，并指定队列类型DISPATCH_QUEUE_CONCURRENT；GCD默认已经提供了全局的并发队列Global queue）
3. 主队列，与主线程功能相同；主线程是唯一可用于更新 UI 的线程。提交至main queue的任务会在主线程中执行。main queue可以调用dispatch_get_main_queue()来获得。因为main queue是与主线程相关的，所以这是一个串行队列。和其它串行队列一样，这个队列中的任务一次只能执行一个。

### 任务
GCD中的任务只是一个代码块，它可以指一个block或者函数指针。根据这个代码块添加进入队列的方式，将任务分为同步任务和异步任务：

1. 同步任务，使用dispatch_sync将任务加入队列。将同步任务加入串行队列，会顺序执行，一般不这样做并且在一个任务未结束时调起其它同步任务会死锁。将同步任务加入并行队列，会顺序执行，但是也没什么意义。
2. 异步任务，使用dispatch_async将任务加入队列。将异步任务加入串行队列，会顺序执行，并且不会出现死锁问题。将异步任务加入并行队列，会并行执行多个任务，这也是我们最常用的一种方式。

用一张图来总结下队列和同步异步的关系：
![](/img/15246478174917.jpg)

## GCD常见用法及场景展示

#### 1. dispatch_async

```
dispatch_queue_t globalQueue = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0);
dispatch_async(globalQueue, ^{
    // 一个异步的任务，例如网络请求，耗时的文件操作等等
    ...
    dispatch_async(dispatch_get_main_queue(), ^{
        // UI刷新
        ...
    });
});
```
这种用法非常常见，比如开启一个异步的网络请求，待数据返回后返回主队列刷新UI；又比如请求图片，待图片返回刷新UI等等。

#### 2. dispatch_after

```
dispatch_after(dispatch_time(DISPATCH_TIME_NOW, (int64_t)(5.0 * NSEC_PER_SEC)), queue, ^{
    // 在queue里面延迟执行的一段代码
    ...
});
```
这为我们提供了一个简单的延迟执行的方式，比如在view加载结束延迟执行一个动画等等。

#### 3. dispatch_once

```
static dispatch_once_t onceToken;
dispatch_once(&onceToken, ^{
    // 只执行一次的任务
    ...
});
```
可以使用其创建一个单例，也可以做一些其他只执行一次的代码

#### 4. dispatch_group

```
dispatch_queue_t queue = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0);
dispatch_group_t group = dispatch_group_create();

dispatch_group_async(group, queue, ^{
    // 异步任务1
});

dispatch_group_async(group, queue, ^{
    // 异步任务2
});

dispatch_group_notify(group, mainQueue, ^{
    // 任务完成后，在主队列中做一些操作
    ...
});
```
上述方式，可以适用于自己维护的一些异步任务的同步问题；但是如果任务是再在队列里开启一个异步任务的话，就不能保证完成任务的同步问题了； 比如任务是网络请求等，我们要同时开启异步任务下载3张图片，下载完毕后，在将三张图片合成；
这里可以用通过一种计数的方式控制任务间同步，dispatch_group_enter和dispatch_group_leave
![](/img/15246492738036.jpg)

模拟网络请求的代码：

```
dispatch_group_enter(group);
[JDApiService getActivityDetailWithActivityId:self.activityId Location:stockAddressId SuccessBlock:^(NSDictionary *userInfo) {
    // 数据返回后一些处理
    ...
    
    dispatch_group_leave(group);
} FailureBlock:^(NSError *error) {
    // 数据返回后一些处理
    ...
    
    dispatch_group_leave(group);
}];

dispatch_group_enter(group);
[JDApiService getAllCommentWithActivityId:self.activityId PageSize:3 PageNum:self.commentCurrentPage SuccessBlock:^(NSDictionary *userInfo) {
    // 数据返回后一些处理
    ...
    
    dispatch_group_leave(group);
} FailureBlock:^(NSError *error) {
    // 数据返回后一些处理
    ...
    
    dispatch_group_leave(group);
}];

执行dispatch_group_notify的任务。
dispatch_group_notify(group, mainQueue, ^{
    // 一般为回主队列刷新UI
    ...
});

```

#### 5. dispatch_barrier_async


```
dispatch_queue_t queue = dispatch_queue_create("queue", DISPATCH_QUEUE_CONCURRENT);
dispatch_async(queue, ^{
    // 任务1
    ...
});
dispatch_async(queue, ^{
    // 任务2
    ...
});
dispatch_async(queue, ^{
    // 任务3
    ...
});
dispatch_barrier_async(queue, ^{
    // 任务4
    ...
});
dispatch_async(queue, ^{
    // 任务5
    ...
});
dispatch_async(queue, ^{
    // 任务6
    ...
});
```
dispatch_barrier_async的作用可以用一个词概括－－承上启下，它保证此前的任务都先于自己执行，此后的任务也迟于自己执行。本例中，任务4会在任务1、2、3都执行完之后执行，而任务5、6会等待任务4执行完后执行。
和dispatch_group类似，dispatch_barrier也是异步任务间的一种同步方式，可以在比如文件的读写操作时使用，保证读操作的准确性。
但是有一点需要注意，dispatch_barrier_sync和dispatch_barrier_async只在自己创建的并发队列上有效，在全局(Global)并发队列、串行队列上，效果跟dispatch_(a)sync效果一样。
官方文档上的说法：
![](/img/15246499280964.jpg)

#### 6. dispatch_apply

```
// for循环做一些事情，输出0123456789
for (int i = 0; i < 10; i ++) {
    NSLog(@"%d", i);
}

// dispatch_apply替换（当且仅当处理顺序对处理结果无影响环境），输出顺序不定，比如1098673452
dispatch_queue_t queue = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0);
/*! dispatch_apply函数说明
 *
 *  @brief  dispatch_apply函数是dispatch_sync函数和Dispatch Group的关联API
 *         该函数按指定的次数将指定的Block追加到指定的Dispatch Queue中,并等到全部的处理执行结束
 *
 *  @param 10    指定重复次数  指定10次
 *  @param queue 追加对象的Dispatch Queue
 *  @param index 带有参数的Block, index的作用是为了按执行的顺序区分各个Block
 *
 */
dispatch_apply(10, queue, ^(size_t index) {
    NSLog(@"%zu", index);
});
```
那么，dispatch_apply有什么用呢，因为dispatch_apply并行的运行机制，效率一般快于for循环的类串行机制（在for一次循环中的处理任务很多时差距比较大）。比如这可以用来拉取网络数据后提前算出各个控件的大小，防止绘制时计算，提高表单滑动流畅性，如果用for循环，耗时较多，并且每个表单的数据没有依赖关系，所以用dispatch_apply比较好。

#### 7. dispatch_suspend和dispatch_resume

```
dispatch_queue_t queue = dispatch_get_main_queue();
dispatch_suspend(queue); //暂停队列queue
dispatch_resume(queue);  //恢复队列queue
```
这两个函数不会影响到队列中已经执行的任务，队列暂停后，已经添加到队列中但还没有执行的任务不会执行，直到队列被恢复。

#### 8. dispatch_semaphore_signal

```
// a、同步问题：输出肯定为1、2、3。
dispatch_queue_t queue = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0);
dispatch_semaphore_t semaphore1 = dispatch_semaphore_create(1);
dispatch_semaphore_t semaphore2 = dispatch_semaphore_create(0);
dispatch_semaphore_t semaphore3 = dispatch_semaphore_create(0);

dispatch_async(queue, ^{
    // 任务1
    dispatch_semaphore_wait(semaphore1, DISPATCH_TIME_FOREVER);
    NSLog(@"1\n");
    dispatch_semaphore_signal(semaphore2);
    dispatch_semaphore_signal(semaphore1);
});

dispatch_async(queue, ^{
    // 任务2
    dispatch_semaphore_wait(semaphore2, DISPATCH_TIME_FOREVER);
    NSLog(@"2\n");
    dispatch_semaphore_signal(semaphore3);
    dispatch_semaphore_signal(semaphore2);
});

dispatch_async(queue, ^{
    // 任务3
    dispatch_semaphore_wait(semaphore3, DISPATCH_TIME_FOREVER);
    NSLog(@"3\n");
    dispatch_semaphore_signal(semaphore3);
});
```

```
// b、有限资源访问问题:for循环看似能创建100个异步任务，实质由于信号限制，最多创建10个异步任务。
dispatch_queue_t queue = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0);
dispatch_semaphore_t semaphore = dispatch_semaphore_create(10);
for (int i = 0; i < 100; i ++) {
    dispatch_semaphore_wait(semaphore, DISPATCH_TIME_FOREVER);
    dispatch_async(queue, ^{
        // 任务
        ...
        dispatch_semaphore_signal(semaphore);
    });
}
```
主要是有两种用法：  1.解决同步问题 2.解决有限资源访问

## 内存和安全
* MRC:用dispatch_retain和dispatch_release管理dispatch_object_t内存。
* ARC:ARC在编译时刻自动管理dispatch_object_t内存，使用retain和release会报错。
* dispatch_queue是线程安全的，你可以随意往里面添加任务。
## 一些经常拿出来说的代码

1. 在串行队列里的异步任务，任务里再同一个队列里异步任务
```
 dispatch_queue_t queue = dispatch_queue_create("com.tzjy.-02--GCD", DISPATCH_QUEUE_SERIAL);    dispatch_async(queue, ^{//block1
        NSLog(@"1");
        NSLog(@"1---%@",[NSThread currentThread]);
        dispatch_async(queue, ^{//block2
            NSLog(@"2");
            NSLog(@"2---%@",[NSThread currentThread]);
        });
        sleep(2);
        NSLog(@"3");
        NSLog(@"3---%@",[NSThread currentThread]);
    });
```
这里执行的结果肯定是：1，3，2  且线程是同一个  因为是串行队列；

2. 在并发队列里的异步任务，任务里再同一个队列里异步任务
```
    dispatch_queue_t globalqueue = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0);
    dispatch_async(queue, ^{//block1
        NSLog(@"1");
        NSLog(@"1---%@",[NSThread currentThread]);
        dispatch_async(queue, ^{//block2
            NSLog(@"2");
            NSLog(@"2---%@",[NSThread currentThread]);
        });
        sleep(2);
        NSLog(@"3");
        NSLog(@"3---%@",[NSThread currentThread]);
    });
```
这里的执行结果是1，2，3 且会有不同的两个线程，因为是并发队列；

3. 在串行队列里同，异步任务再同步任务，会发生死锁

```
    dispatch_queue_t queue = dispatch_queue_create("com.tzjy.-02--GCD", DISPATCH_QUEUE_SERIAL);
    dispatch_async(queue, ^{//block1
        NSLog(@"1");
        dispatch_sync(queue, ^{//block2
            NSLog(@"2");
        });
        sleep(2);
        NSLog(@"3");
        
    });
```
这里会崩溃掉，因为任务一会等待任务二，而任务二又会等待任务一，这样形成死锁；如果这里的队列不是串行队列，是并行队列，则不会死锁。

> 如果使用dispatch_sync，如果执行dispatch_sync 这句的上下文环境的“队列”，跟 dispatch_sync(queue, ^{ 的这个queue 是同一个队列，不用想了绝逼死锁。。。当然还有一个条件，这个队列必须是单行车道，也就是串行队列，如果是并行的，不会有死锁，为什么呢？

> 假如queue是个并行队列，当执行到dispatch_sync 的时候，dispatch_sync无需等待当前任务执行完毕，此时它就会执行，但是会阻塞当前线程，只有等它执行完了，下面的任务才会接着去执行了。


