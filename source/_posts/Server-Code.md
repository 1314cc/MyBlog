---
title: 服务器编程框架
date: 2017-02-21 08:16:55
tags: [服务器 , 高并发]
category: [服务器编程]
comments:
---
服务器编程框架
======================

## 两种高效的事件处理模式
 同步I/O模型通常用于实现` Reactor `,异步I/O模式则用于实现` Proactor `模式,不过用同步I/O方式,也可以模拟出` Proactor` 模式

### Reactor
`Reactor`,要求主线程,只负责监听文件描述符上是否有事件发生,有的话就立即将该时间通知工作线程,除此之外,主线程不做任何其他实质性的工作,读写数据,接受新链接,以及处理客户请求均在工作线程中完成.

**工作流程:**

- 主线程往`epoll`内核事件表中注册socket上的`读`就绪事件.
- 主线程调用`epoll_wait`等待socket上有数据可读.
- 当socket上有数据可读时,`epoll_wait`通知主线程,主线程则将socket可读事件放入请求队列.
- 睡眠在请求队列上的某个工作线程被唤醒,它从socket中读取数据,并处理客户请求,然后往epoll内核事件表中注册该socket上的写就绪事件.
- 主线程调用epoll_wait等待socket可写.
- 当socket可写时,` epoll_wait`通知主线程,主线程将socket可写事件放入请求队列.
- 睡眠在请求队列上的某个工作线程被唤醒,它往socket上写入服务器处理客户请求的结果.


### Proactor

**与Reactor模式不同,Proactor将所有的I/O操作都交给主线程和内核来处理,工作线程仅仅负责业务逻辑.**

使用异步I/O模型(以`aio_read 和 aio_write`为例)

**工作流程:**

 - 主线程调用`aio_read`函数想内核注册socket上的**读完成**事件,并告诉内核用户读缓冲区的位置,以及读操作完成后如何通知应用程序(以信号为例)
 - 主线程继续处理其他逻辑
 - 当socket上的数据被读入用户缓冲区后,内核将向应用程序发送一个信号,以通知应用程序数据可用.
 - 应用程序预先定义好的信号处理函数选择一个工作线程来处理客户请求.工作线程处理完客户请求之后,调用`aio_write`函数向内核注册socket上的写完成事件,并告诉内核用户写缓冲区的位置,以及写操作完完成时如何通知应用程序.
 - 主线程继续处理其他逻辑.
 - 当用户缓冲区的数据被写入socket中后,内核将向应用程序发送一个信号,以通知应用程序数据已经发送完毕.
 - 应用程序预先定义好的信号处理函数选择一个工作线程来做善后处理,比如决定是否关闭socket.
 
### Reactor与Proactor 总结:
Proactor将所有的I/O操作交给主线程和内核来处理,工作线程只是负责业务处理, Reactor是通知的是**可读可写**,而Proactor是通知**读完成**和**写完成**

## 两种高效的并发模式
并发编程的目的是让程序"同时"执行多个任务. 如果程序是计算密集型的,并发编程并没有优势,反而由于任务的切换而效率降低. 但是如果程序是I/O密集型的,比如经常读写文件,访问数据库等,则情况就不同了. 由于I/O的速度远远没有CPUs计算速度快,所以让程序阻塞与I/O操作将浪费大量的CPU时间.如果程序有多个执行线程.则当前被I/O操作所阻塞的执行线程可主动放弃CPU,并将执行权限转移到其他线程,这样一来,CPU可用来做更加有意义的事情,而不是等待I/O操作完成,因此CPU的利用率明显上升.
** 并发模式:**
 - 半同步/半异步 模式
 - 领导者/追随者 模式
### 半同步/半异步模式
> 此时的同步异步,和I/O模型中的同步.异步是不同的概念. 在I/O模型中,"同步"和"异步"区分的是内核向应用程序通知的是何种I/O事件(`就绪事件 和 完成事件`),已经该由谁来完成I/O读写(`应用程序还是内核`). 在并发模式中,"同步"指定是程序完全`按照代码序列的顺序执行`;"异步"指的是程序的`执行需要由系统事件来驱动`. 常见的系统事件包括中断,信号等.

半同步/半异步模式中,同步线程用于处理客户逻辑;异步线程用于处理处理I/O事件. 异步线程监听到客户请求之后,就将其封装成请求对象并插入请求队列当中.请求队列将通知某个工作在同步模式的工作线程来读取并处理该请求对象.

#### 半同步/半反应堆模式
这种模式是结合两种事件处理方式(Reactor,Proactor)和几种I/O模型模型变体中的一种.
异步线程只有一个,线程充当,他负责监听所有SOCKET上的事件.如果监听socket上有可读事件,即有新的链接过来,主线程就接收之已得到新的链接socket,然后往epoll内核事件表中注册该socket上的读写事件,如果链接socket上有读写事件发生,即有新的客户请求到来或这有数据要发送到客户端,主线程就将该链接socket插入请求队列中.所有的工作线程都睡眠在请求队列上,当有任务到来时,他们通过竞争获得任务接管权.

> 缺点:
1. 主线程和工作线程共享请求队列.主线程往请求队列中添加任务,或者工作线程从请求队里中取任务,都要加锁保护,**白白浪费CPU时间**.
2. 每个工作线程同一时间只能处理一个任务,如果客户请求较多,而工作线程较少,则请求队列中堆积很多任务对象,客户端响应越来越慢,如果增加工作线程,则工作线程的切换也白白浪费CPU时间.

#### 高效的半同步/半异步模式
主线程监听socket,链接socket由工作线程来管理.当有新的链接到来时,主线程接受并将返回的链接socket派发给某个工作线程,此后该新socket上的任何I/O操作都由被选中的工作线程来处理,知道客户关闭链接.主线程向工作线程派发socket使用**管道**,工作线程检测管道上有数据可读时,判断是否是新的客户链接请求,如果是,则把该新的socket上的读写时间注册到自己的epoll内核事件表中.

#### 总结:
1. 半同步/半异步 与 I/O模型中的同步异步是不同概念.
2. 半同步/半反应堆
3. 高效的半同步半异步 -- 主线程和工作线程分别有自己的epoll内核事件,分别处理链接和读写.

