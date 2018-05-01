---
title: 01 Netty — 异步和事件驱动
date: 2018-02-08 13:17:35
tags: [笔记,Netty,Java] 
categories: Netty
---
点击查看 [《Netty in Action》笔记目录](https://mindawei.github.io/2018/03/01/%E3%80%8ANetty-in-Action%E3%80%8B%E7%AC%94%E8%AE%B0%E7%9B%AE%E5%BD%95/)。

本文是对[《Netty in Action》](https://download.csdn.net/download/u010738033/10245651)第1章内容的笔记和翻译，主要内容包括：
* Java 中的网络使用
* 介绍 Netty
* Netty 的核心组件

<!-- more -->

Netty 是一个基于异步的、事件驱动的网络应用开发框架，可以为频繁的开发和维护提供方便，并为客户端和服务端提供高性能的协议。

Netty 不仅仅是一个网络框架，它的整体架构和设计思想和它的技术内容一样重要. 为此，我们将要讨论以下一些要点：
* 将关注点隔离（将业务逻辑和网络逻辑进行解耦）
* 模块化和可重用性
* 第一需求是系统可以被测试

## Java 中的网络编程

Java 中传统的网络代码实现如下所示：
![](/images/00009/NetworkingInJava.png)

上面代码背后的处理模型如下所示：
![使用阻塞 I/O 的多个连接](/images/00009/01.png "使用阻塞 I/O 的多个连接")

上面模型的缺点是：
* 浪费资源：在任何一个时刻，都可能由于等待输入或输出的阻塞，造成很多线程休眠。 
* 很难支持大量数目的线程，因为任何一个线程都需要分配栈内存，而计算机内存是有限的。
* 线程间的上下文切换会严重影响性能。

### Java NIO
本地 Socket 库早就已经包含了非阻塞的调用方式。通过这些方法，我们可以对网络资源的使用进行更加多的控制。
* 通过使用 `setsockopt()`，你可以配置 Socket，使得当没有数据的时候，读写调用可以马上返回。
* 你可以注册一系列的非阻塞 Socket，并通过系统的事件通知 API 来了解读写的数据是否已经准备好。

2002年的时候，Java 组织在 JDK 1.4 版本中的 `java.nio` 包中引入了非阻塞的 IO（non-blocking I/O）。

> __新 IO 还是非阻塞 IO ?__
NIO 一开始是 New Input/Output 的缩写, 但是 Java API 已经存在很长的时间了，它已经不再被认为是新的 API 了。很多使用者认为 NIO 带表了非阻塞 IO（non-blocking I/O），而原来的阻塞 IO（blocking I/O）则被认为是 OIO 或 old input/output。

### Selectors
`java.nio.channels.Selector` 类是 Java 非阻塞 IO 实现的核心，它使用了__事件驱动 API__ 来表明哪些非阻塞 sockets 已经准备好可以进行 I/O 了。下图展示了：在这样的模型下，一个单线程可以处理多个并发的连接。

![基于 Selector 的非阻塞 IO](/images/00009/02.png "基于 Selector 的非阻塞 IO")

和阻塞型 IO 模型相比，这样的模型提供了更好的资源管理：
* 连接可以被更少的线程支持，因此在内存管理和上下文切换上有更少的开销。
* 当本任务没有 IO 处理的时候，线程可以重新绑定到其它任务上。

## Netty 介绍
直接使用相对原生的 API，会增加开发的复杂度，并要求开发者具备很高的开发技能，但这样的人才还是很紧缺的。因此，需要采用一个面向对象的基本思想： __隐藏复杂的实现，提供简单的抽象__.

<style>
table th:first-of-type {
    width: 15%;
}
</style>

Netty 的特性总结: 

类别| Netty 特性|
----|----
设计 | 对多种传输类型统一了 API，包括阻塞和非阻塞。<br>简单但高效的线程模型。<br>支持无连接数据报套接字。<br>逻辑组件的连接，支持组件重用。<br>
易用性 | 完善的 Javadoc 以及大量使用实例。<br>只需要依赖 JDK 1.6+。（一些可选的特性依赖 JDK 1.7+ 或者一些额外的依赖)
性能 | 和 Java 本身的 API 相比，具备更高的吞吐量和更低的延时。<br>通过池化和重用，减少了资源的消耗。<br>最小化内存拷贝。
鲁棒性 	| 不会由于连接的快、慢、负载较大而导致 `OutOfMemoryError`。<br>在高速网络场景中，消除了 NIO 应用程序典型的不公平读写比。
安全性 | 支持 SSL/TLS 、StartTLS。<br>在 Applet 或者 OSGI 等资源受限环境中也可以使用。
社区驱动 |  发布频繁和活跃。

异步和事件驱动的优点： 
* 非阻塞的网络调用，使得我们从等待操作完成的过程中解放出来。基于这个特性的异步 IO 使得我们可以实现：异步方法立即返回，当完成的时候可以立刻或者稍后通知我们。
* `Selector` 允许我们通过更少的线程来监控更多的连接和事件。

## Netty 的核心组件
在这一节，我们要讨论 Netty 主要的构建模块：
* `Channels`
* `Callbacks`
* `Futures`
* `Events` 和 `Handlers`

这些构建模块代表了不同类型的构建要素: 资源、逻辑和通知。应用程序将会通过这些模块来访问网络和数据。

### Channels
Channel 是 Java NIO 的基本构成。它表示：
> 与实体间打开的一个连接。这些实体包括：一个硬件设备、一个文件、一个网络 socket、或者一个程序组件，它们可以进行不同的 I/O 操作，例如：读写操作。

Channel 可以被打开、关闭、建立连接、取消连接。

### Callbacks
Callback 是一个回调方法, 本质上是指向其它方法的一个引用。通过回调可以实现：在合适时，调用指向的函数。

Netty 内部使用 callbacks 来处理事件。当 callback 被触发，事件可以被对应的 `ChannelHandler` 接口实现来处理。下面的图片展示了这样的一个例子：当一个新的连接建立后，`ChannelHandler` 会回调 `channelActive()` 方法，该方法将会把消息打印出来。
![](/images/00009/03.png)

### Futures
`Future` 提供了另一种通知机制来让应用知道操作已经完成。

JDK 中提供了 `java.util.concurrent.Future` 接口，但是提供的实现实现只能允许你：手工判断操作是否完成、或者阻塞到操作完成。这样的机制不是很方面，所以 Netty 提供了自己的实现 `ChannelFuture`，它可以实现异步操作的执行。

`ChannelFuture` 提供了一个额外的方法，可以允许我们注册一个或多个 `ChannelFutureListener` 实例。这些监听者的回调函数 `operationComplete()` 会在方法完成时被调用。

Netty 的每个输出 I/O 操作会返回一个 `ChannelFuture`，也就是说，这些输出操作是非阻塞的，会立即返回。 

下例中 `ChannelFuture` 作为 I/O 操作返回的一部分。其中 `connect()` 将会没有阻塞立即返回，而调用的方法会在后台完成。 
![](/images/00009/04.png)

下例展示了如何使用 `ChannelFutureListener`。 
![](/images/00009/05.jpg)

`ChannelFutureListener` 是一个更加完善的 callback。事实上，callbacks 和 Futures 是互补的，它们是 Netty 的一个重要组成部分。

### Events and handlers
Netty 使用不同的事件来告知我们：状态的改变、操作处于哪个阶段。 对应的行为可能包括：
* 日志
* 数据传输
* 流控
* 应用逻辑

Netty 是一个网络框架，所以事件可以根据它们在输入还是输出流进行区分。

输入数据或者输入相关状态改变的事件包括：
* 连接的有效和无效
* 数据读取
* 用户事件
* 错误事件

输出事件是相关操作的返回结果，它将会在未来的某个时刻被调用，包括：
* 打开或者关闭一个远程连接。
* 在一个 socket 中写入或者刷新数据。

![在 ChannelHandlers 链中的输入或者输出事件](/images/00009/06.png "在 ChannelHandlers 链中的输入或者输出事件")

目前，你可以把每个 handler 实例看做是针对特定的事件（event）的一个回调（callback）。

### 结合以上要素
#### Futures、Callbacks、Handlers
Netty 的异步编程模型是基于 `Futures` 和 callbacks 建立的, 而将事件分发到对应的处理 handler 这样的操作是封装在 Netty 内部较深层次的。总而言之，这些元素提供了一个处理环境：可以使得应用的业务逻辑和网络的具体操作进行解耦。__这是 Netty 设计的根本目标。__

只要提供 callbacks 或者使用操作返回的 Futures，就可以实现在程序运行时拦截操作和转移输入输出数据。__这使得链接操作变得非常容易和高效，并能促进写可重用、泛化程度更高的代码。__

#### Selectors、Events、Event Loops
在内部，EventLoop 被绑定到每个 Channel 上，来处理所有的事件（events），包括：
* 重新注册感兴趣的事件
* 分发事件到 ChannelHandlers
* 调度下一步的响应

__EventLoop 本身只被一个线程驱动，处理所绑定的 Channel 的所有 I/O 事件，并且这种关系在 EventLoop 的生命周期中不会改变。__ 这个简单和强大的设计可以消除你对同步 ChannelHandlers 的担心。

