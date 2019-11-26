# netty NioEventLoopGroup

## NioEventLoopGroup线程数

```java
DEFAULT_EVENT_LOOP_THREADS = Math.max(1, SystemPropertyUtil.getInt(
                "io.netty.eventLoopThreads", Runtime.getRuntime().availableProcessors() * 2));
```

可以通过io.netty.eventLoopThreads这个系统参数在程序变量中设置，也给以在NioEventLoopGroup new一个实例的时候指定线程数，如果不指定线程数默认的是 获取服务器的cpu核数乘以2.

> [英特尔](https://baike.baidu.com/item/英特尔/305730)[超线程技术](https://baike.baidu.com/item/超线程技术)   是全新英特尔酷睿 i7 , 酷睿 i5 处理器和英特尔[至强](https://baike.baidu.com/item/至强)5500 系列处理器所具有的一种性能特点。简单来说，它可使处理器中的1 颗内核如2 颗内核那样在操作系统中发挥作用。这样一来，操作系统可使用的执行资源扩大了一倍，大幅提高了系统的整体性能。
>
> 在谈论内核、线程、[超线程](https://baike.baidu.com/item/超线程/86034)这些术语时容易让人产生混淆。为简化概念，1 个内核就是 1 个CPU。每个英特尔[酷睿](https://baike.baidu.com/item/酷睿)i7 或英特尔至强5500 处理器在出厂时均有4 个内核（未来可能会提供其他版本）。

所以如果计算机时4核的，那么默认的线程数时4乘2乘2等于16个线程。

## Channels，EventLoops，Threads 和 EventLoopGroups 的关系：

- 一个EventLoopGroup包含一个或多个EventLoop
- 一个EventLoop被绑定到一个单线程上，在这个EventLoop的整个生命周期。
- 所有的I/O事件处理通过一个EventLoop在一个专门的线程上被处理。
- 一个Channel的整个生命周期里只会被注册到一个EventLoop
- 一个EventLoop可能被分配给一个或多个Channel

给定Channel的I/O会在同一个Thread上执行，实质上这消除了同步的必要。

## ChannelFuture接口

Netty中的所以I/O操作都是异步的。因为操作可能立即返回，之后我们需要一个方式去检测这个操作的返回。为了这个目的，Netty提供了ChannelFuture，ChannelFuture的addListener()方法可以注册一个ChannelFutureListener，该ChannelFutureListener在操作完成( 无论操作成功与否 )时将被通知。

## 线程模型

![](..\img\reactor.png)

Netty的线程模型被称为Reactor模型，具体如图所示，图上的mainReactor指的就是`bossGroup`，这个线程池处理客户端的连接请求，并将accept的连接注册到subReactor的其中一个线程上；图上的subReactor当然指的就是`workerGroup`，负责处理已建立的客户端通道上的数据读写；图上还有一块ThreadPool是具体的处理业务逻辑的线程池，一般情况下可以复用subReactor,但官方建议处理一些较为耗时的业务时还是要使用单独的ThreadPool。

## NioEventLoop

① NioEventLoop是一个基于JDK NIO的异步事件循环类，它负责处理一个Channel的所有事件在这个Channel的生命周期期间。

② NioEventLoop的整个生命周期只会依赖于一个单一的线程来完成。一个NioEventLoop可以分配给多个Channel，NioEventLoop通过JDK Selector来实现I/O多路复用，以对多个Channel进行管理。

③ 如果调用Channel操作的线程是EventLoop所关联的线程，那么该操作会被立即执行。否则会将该操作封装成任务放入EventLoop的任务队列中。

④ 所有提交到NioEventLoop的任务都会先放入队列中，然后在线程中以有序(FIFO)/连续的方式执行所有提交的任务。

⑤ NioEventLoop的事件循环主要完成了：a)已经注册到Selector的Channel的监控，并在感兴趣的事件可执行时对其进行处理；b)完成任务队列(taskQueue)中的任务，以及对可执行的定时任务和周期性任务的处理(scheduledTaskQueue中的可执行的任务都会先放入taskQueue中后，再从taskQueue中依次取出执行)。