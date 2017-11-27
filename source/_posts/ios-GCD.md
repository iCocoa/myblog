---
title: Grand Central Dispatch
date: 2015-01-05 20:13:18
tags: iOS
---

**GCD**（Grand Central Dispatch）是异步执行任务的技术之一。一般将应用程序中记述的线程管理用的代码在系统级中实现。开发者只需要定义想执行的任务并追加到适当的 Dispatch Queue 中，GCD 就能生成必要的线程并计划执行任务。由于线程管理是作为系统的一部分来实现的，因此可统一管理，也可执行任务，这样就比以前的线程更有效率。

```
dispatch_queue_t queue = dispatch_queue_create("myQueue", DISPATCH_QUEUE_CONCURRENT);
dispatch_async(queue, ^{
   /**
    *  长时间处理
    *  例如：AR用图像识别、数据库访问
    */
   
   /**
    *  长时间处理结束，主线程使用该处理结果
    */
   
   dispatch_async(dispatch_get_main_queue(), ^{
       /**
        *  只在主线程可以执行的处理
        *  例如用户界面刷新
        */
   });
});
```

在导入 GCD 之前，Cocoa 框架提供了 NSObject 类的`performSelectorInBackground:withObject`实例方法和`performSelectorOnMainThread`实例方法等简单的多线程编程技术。

**线程**

	线程是程序中一个单一的顺序控制流程。进程内一个相对独立的、可调度的执行单元，是系统独立调度和分派 CPU 的基本单位。在单个程序中同时运行多个线程完成不同的工作，称为多线程。
	“一个 CPU 执行的 CPU 命令列为一条无分叉路径”即为“线程”。
	现在一个物理的 CPU 芯片实际上有64个（64核）CPU，尽管如此，“一个 CPU 执行的 CPU 命令列为一条无分叉路径”仍然不变。
		
	OS X 和 iOS 的核心 XNU 内核在发生操作系统事件时（如每隔一定时间，唤起系统调用等情况）会切换执行路径。执行中路径的状态，例如CPU的寄存器等信息保存到各自路径专用的内存块中，从切换目标路径专用的内存块中，复原 CPU 寄存器等信息，继续执行切换路径的 CPU 命令列。这称为“上下文切换”。
	由于使用多线程的程序可以在某个线程和其他线程之间反复多次进行上下文切换，因此看上去好像1个 CPU 核能够并列地执行多个线程一样。而且在具有多个 CPU 核的情况下，就不是“看上去像”了，而是真的提供了多个CPU核并行执行多个线程的技术。

**使用多线程容易引发的常见问题**

* 数据竞争（多个线程更新相同的资源会导致数据不一致）
* 死锁（停止等待事件的线程会导致多个线程相互持续等待）
* 内存占用（使用太多线程会消耗大量内存）

尽管容易发生问题，但是为了保证应用程序的响应性能，也应当使用多线程编程。


### GCD的API

Dispatch Queue是执行处理的等待队列，按照FIFO（先进先出）的追加顺序执行处理。开发者要做的只是定义想执行的任务并追加到适当地Dispatch Queue中。
Dispatch Queue分两种：

* 等待现在执行中处理结束的Serial Dispatch Queue;
* 不等待现在执行中处理结束的Concurrent Dispatch Queue。


**dispatch\_queue\_create**
	
生成Dispatch Queue的方法。

`dispatch_queue_t queue = dispatch_queue_create("com.example.gcd.MyQueue", DISPATCH_QUEUE_CONCURRENT);`
	
`/* dispatch_release(queue); */`

如果你部署的最低目标低于 iOS 6.0 or Mac OS X 10.8 ，你应该自己管理GCD对象,使用(dispatch_retain,dispatch_release),ARC并不会去管理它们。如果你部署的最低目标是 iOS 6.0 or Mac OS X 10.8 或者更高的，
 ARC 已经能够管理 GCD 对象了,这时候, GCD 对象就如同普通的 OC 对象一样,不应该使用 dispatch\_retain 或者 dispatch\_release 。

为了避免多个线程更新相同资源导致数据竞争，推荐使用 Serial Dispatch Queue。当想并发执行不发生数据竞争等问题的处理时，使用 Concurrent Dispatch Queue。
	
**Main Dispatch Queue / Global Dispatch Queue**
	
系统提供的 Dispatch Queue。
	
Main Dispatch Queue 是在主线程中执行的 Dispatch Queue。
因为主线程只有1个，所以它是 Serial Dispatch Queue。
追加到 Main Dispatch Queue 的处理在主线程的 Runloop 中执行。
	
**Global Dispatch Queue**是所有应用程序都能够使用的 Concurrent Dispatch Queue。没有必要通过 dispatch\_queue\_create 函数逐个生成 Concurrent Dispatch Queue。只要获取 Global Dispatch Queue 使用即可。
	
表 1-1 Dispatch Queue种类
	
|	名称        	    | Dispatch Queue的种类 |     说明     |
| :-------------:  |   :-------------:   | :-----------:| 
|Main Dispatch Queue|Serial Dispatch Queue| 主线程执行|
|Global Dispatch Queue(High Priority)|Concurrent Dispatch Queue|执行优先级：高（最高优先级）|
|Global Dispatch Queue(Default Priority)|Concurrent Dispatch Queue|执行优先级：默认|
|Global Dispatch Queue(Low Priority)|Concurrent Dispatch Queue|执行优先级：低|
|Global Dispatch Queue(Background Priority)|Concurrent Dispatch Queue|执行优先级：后台|

获取方法：

```
dispatch_queue_t mainDispatchQueue = dispatch_get_main_queue();
dispatch_queue_t globalDispatchQueueHigh = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_HIGH, 0);
```
    	
**dispatch\_set\_target\_queue**

dispatch\_queue\_create 函数生成的 Dispatch Queue 不管是 Serial Dispatch Queue 还是 Concurrent Dispatch Queue，都使用与默认优先级 Global Dispatch Queue 相同执行优先级的线程。而变更生成的 Dispatch Queue 的执行优先级要使用 dispatch\_set\_target\_queue 函数。

```	
dispatch_queue_t mySerialDispatchQueue = dispatch_queue_create("com.example.gcd.mySerialDispatchQueue", NULL);
dispatch_queue_t globalDispatchQueueBackground = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_BACKGROUND, 0);
dispatch_set_target_queue(mySerialDispatchQueue, globalDispatchQueueBackground);
```
	
指定要变更执行优先级的 Dispatch Queue 为 dispatch\_set\_target\_queue 函数的第一个参数，指定与要使用的执行优先级相同优先级的 Global Dispatch Queue 为第二个参数（目标优先级）。
		
**dispatch_after**

当我们想要在指定时间后执行某个处理时（切确来说，是在指定时间追加处理到Dispatch Queue），使用 dispatch\_after 函数。
	
```
/* 在3秒后用dispatch_asyn函数追加Block到Main Dispatch Queue 
 * ull 是C语言的数值字面量，是显示表明类型时使用的字符串（表示“unsigned long long”）
 */
dispatch_time_t time = dispatch_time(DISPATCH_TIME_NOW, 3ull * NSEC_PER_SEC);
dispatch_after(time, dispatch_get_main_queue(), ^{
  NSLog(@"waited at least three seconds.");
});
```
    	
第一个参数是指定时间用的 dispatch\_time\_t 类型的值。可以使用 dispatch\_time 函数或者 dispatch\_walltime 函数获得。 dispatch\_time 函数能够获取从第一个参数 dispatch\_time\_t 类型值中指定的时间开始，到第二个参数指定的毫微秒单位时间后的时间。（相对时间） dispatch\_walltime 函数通常用于计算绝对时间，比如：2011年11月11日11分11秒 这一绝对时间，这可以当做粗略的闹钟功能使用。 dispatch\_walltime 函数由 POSIX 中使用的 struct timespec 类型的时间得到 dispatch\_time\_t 类型的值。

**Dispatch Group**

在追加到 Dispatch Queue 中的多个处理全部结束后想执行结束处理，这种情况会经常出现。只使用一个 Serial Dispatch Queue 时，只要将想执行的处理全部追加到该 Serial Dispatch Queue 中并在最后追加结束处理，即可实现。但是在使用 Concurrent Dispatch Queue 时或同时使用多个 Dispatch Queue时，源代码就会变得颇为复杂。这是就用到 Dispatch Group。

```	
dispatch_queue_t queue = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0);
dispatch_group_t group = dispatch_group_create();
dispatch_group_async(group, queue, ^{
  	NSLog(@"blk1");
});
dispatch_group_async(group, queue, ^{
  	NSLog(@"blk2");
});
dispatch_group_async(group, queue, ^{
  	NSLog(@"blk3");
});
dispatch_group_notify(group, dispatch_get_main_queue(), ^{
  	NSLog(@"done");
});
```
    	
这里除了使用 dispatch\_group\_notify，还可以使用 dispatch\_group\_wait 函数。

```    		
例如：	
long result = dispatch_group_wait(group,DISPATCH_TIME_FOREVER);
永远等待下去，直到全部处理完成，所以result恒为0
	
例如：
dispatch_time_t time = dispatch_time(DISPATCH_TIME_NOW, 1ull * NSEC_PER_SEC);
long result = dispatch_group_wait(group, time);

if (result == 0) {
	// 属于Dispatch Group的全部处理结束
} else {
	// 属于Dispatch Group的某个处理还在执行中
}
```

如果 dispatch\_group\_wait 函数的返回值不为 0，就意味着虽然经过了指定的时间，但属于 Dispatch Group 的某一个处理还在执行中。如果返回值为 0，那么全部处理执行结束。指定`DISPATCH_TIME_NOW`，则不用任何等待即可判定属于 Dispatch Group 的处理是否执行结束。

 `long result = dispatch_group_wait(group,DISPATCH_TIME_NOW);`
   
在主线程的 Runloop 的每次循环中，可检查执行是否结束，从而不耗费多余的等待时间，虽然这样可以，但一般在这种情况下，还是推荐用 dispatch\_group\_notify 函数追加技术处理到 Main Dispatch Queue 中，这样可以简化源代码。
   
**dispatch\_barrier\_async**

在访问数据库或文件时，使用 Serial Dispatch Queue 可避免数据竞争的问题。写入处理确实不可与其他的处理以及包含读取处理的其他处理并行执行。但是如果读取处理只是与读取处理并行执行，那么多个并行执行就不会发生问题。也就是说，为了高效率地进行访问，读取处理追加到 Concurrent Dispatch Queue 中，写入处理在任何一个读取处理没有执行的状态下，追加到 Serial Dispatch Queue 中即可（在写入处理结束之前，读取处理不可执行）。
	
这时，用到`dispatch_barrier_async`函数。`dispatch_barrier_async`函数会等待追加到 Concurrent Dispatch Queue 上的并行执行的处理全部结束之后，再将指定的处理追加到该 Concurrent Dispatch Queue 中。然后在由 dispatch\_barrier\_async 函数追加的处理执行完毕后，Concurrent Dispatch Queue 才恢复为一般的动作，追加到该 Concurrent Dispatch Queue 的处理又开始并行执行。

```	
dispatch_async(queue, blk0_for_reading);
dispatch_async(queue, blk1_for_reading);
dispatch_async(queue, blk2_for_reading);
dispatch_async(queue, blk3_for_reading);

//写入处理
dispatch_barrier_async(queue, blk_for_writing);

dispatch_async(queue, blk4_for_reading);
dispatch_async(queue, blk5_for_reading);
dispatch_async(queue, blk6_for_reading);
```

**dispatch_sync(同步)**
**dispatch_apply(同步操作)**

dispatch\_apply 函数是 dispatch\_sync 函数和 Dispatch Group 的关联API。该函数按指定的次数将指定的 Block 追加到指定的 Dispatch Queue 中，并等待全部处理执行结束。


```	
dispatch_queue_t queue = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0);
dispatch_apply(10, queue, ^(size_t index) {
 	NSLog(@"%zu",index);
});
```
   	
**dispatch\_suspend / dispatch\_resume**

当追加大量处理到 Dispatch Queue 时，在追加处理的过程中，有时希望不执行已经追加的处理。这时，需要挂起 Dispatch Queue，当可以执行时再恢复。
	
dispatch\_suspend 函数挂起指定的 Dispatch Queue。
			
`dispatch_suspend(queue);`
			
dispatch_resume 函数恢复指定的 Dispatch Queue。
		
`dispatch_resume(queue);`
			
这两个函数对已经执行的处理没有影响。挂起后，追加到 Dispatch Queue 中但尚未执行的处理在此之后停止执行。而恢复则使得这些处理能够继续执行。

**Dispatch Semaphore**

如前所述，当并行执行的处理更新数据时，会产生数据不一致的情况，有时应用程序还会异常结束。虽然使用 Serial Dispatch Queue 和 dispatch\_barriel\_async 函数可以避免这类问题，但有必要进行更细粒度的排他控制。
	
Dispatch Semaphore 是持有计数的信号，该计数是多线程编程中的计数类型信号。计数为 0 时等待，计数为 1 或者大于 1 时，减去 1 而不等待。
	
```
dispatch_queue_t queue = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0);
/**
*  生成Dispatch Semaphore
*
*  Dispatch Semaphore的计数初始值设定为“1”。
*
*  保证可访问 NSMutableArray 类对象的线程同时只能有一个
*
*/
dispatch_semaphore_t semaphore = dispatch_semaphore_create(1);
    
NSMutableArray *array = [NSMutableArray array];
    
for (NSInteger i = 0; i < 100000; i++) {
   	dispatch_async(queue, ^{
       /**
        * 等待 Dispatch Semaphore
        * 一直等待，直到 Dispatch Semaphore 的计数值大于或者等于 1
        */
       dispatch_semaphore_wait(semaphore, DISPATCH_TIME_FOREVER);
       
       /**
        * 由于 Dispatch Semaphore 的计数值大于等于1
        * 所以将 Dispatch Semaphore 的计数值减去1
        * dispatch_semaphore_wait 函数执行返回
        *
        * 执行到此时，计数值恒为“0”
        *
        * 此时可访问 NSMutableArray 类对象的线程只有1个
        * 可以安全地进行更新
        */
       
       [array addObject:[NSNumber numberWithInt:1]];
       
       /**
        *  排他控制结束
        *  使用 dispatch_semaphore_signal 函数将计数值加 1
        *
        *  如果有通过 dispatch_semaphore_wait 函数等待计数值增加的线程
        *  由最先等待的线程执行。
        */
       
       dispatch_semaphore_signal(semaphore);
   	});
}
```
    	
**dispatch_once**
	
dispatch\_once 函数是保证在应用程序执行中只执行一次指定处理的 API。在生成单例对象时使用。在多线程下执行，也可保证百分之百安全。

```	
static dispatch_once_t pred;
dispatch_once(&pred, ^{
   /**
    *  初始化
    */
});
```
	    
**Dispatch I/O**

在读取较大文件时，如果将文件分成合适的大小使用 Global Dispatch Queue 并列读取的话，会比一般的读取速度快不少。苹果中使用 Dispatch I/O 和  Dispatch Data 的例子
	
```	
pipe_q = dispatch_queue_create("PipeQ", NULL);
pipe_channel = dispatch_io_create(DISPATCH_IO_STREAM, fd, pipe_q, ^(int err){
   close(fd);
});
    
*out_fd = fdpair[1];
    
dispatch_io_set_low_water(pipe_channel, SIZE_MAX);
    
dispatch_io_read(pipe_channel, 0, SIZE_MAX, pipe_q, ^(bool done, dispatch_data_t pipedata, int err){
   if (err == 0)
   {
       size_t len = dispatch_data_get_size(pipedata);
       if (len > 0)
       {
           const char *bytes = NULL;
           char *encoded;
           dispatch_data_t md = dispatch_data_create_map(pipedata, (const void **)&bytes, &len);
           encoded = asl_core_encode_buffer(bytes, len);
           asl_set((aslmsg)merged_msg, ASL_KEY_AUX_DATA, encoded);
           free(encoded);
           _asl_send_message(NULL, merged_msg, -1, NULL);
           asl_msg_release(merged_msg);
           dispatch_release(md);
       }
   }
   
   if (done)
   {
       dispatch_semaphore_signal(sem);
       dispatch_release(pipe_channel);  
       dispatch_release(pipe_q);  
   }  
});
```
	
以上摘自 Apple System Log API 用的源代码(Libc-763.11 gen/asl.c)。`dispatch_io_create`函数创建了一个 dispatch I/O。并指定发生错误的时候被执行的block，以及执行该 block 的队列。`dispatch_io_set_low_water`函数设定一次读取的大小（分割大小），`dispatch_io_read`函数在全局队列上开启读取操作。每当一块数据被读取后，数据作为参数会被传递给`dispatch_io_read`函数指定的读取结束时回调的 Block。回调的用的 Block 分析传递过来的 Dispatch Data 并进行结合处理。
