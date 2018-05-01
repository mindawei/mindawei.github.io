---
title: 03 Netty的组件和设计
date: 2018-02-10 18:04:17
tags: [笔记,Netty,Java] 
categories: Netty
---

点击查看 [《Netty in Action》笔记目录](https://mindawei.github.io/2018/03/01/%E3%80%8ANetty-in-Action%E3%80%8B%E7%AC%94%E8%AE%B0%E7%9B%AE%E5%BD%95/)。

本文是对[《Netty in Action》](https://download.csdn.net/download/u010738033/10245651)第3章内容的笔记和翻译，主要内容包括：
* Netty 技术和架构方面的介绍
* `Channel`、`EventLoop` 和 `ChannelFuture`
* `ChannelHandler` 和 `ChannelPipeline`
* 启动（ Bootstrapping）

<!-- more -->

## Channel、EventLoop 和 ChannelFuture
下面这些可以被看做 Netty 网络模型的抽象：
* `Channel`：`Sockets`
* `EventLoop`：流控、多线程、并发
* `ChannelFuture`：异步通知

### Channel 接口
Java 中基本的网络 I/O 操作依赖 `Socket` 类，如：`bind()`、`connect()`、`read()` 和 `write()`。Netty 中的 `Channel` 接口提供的 API 与直接操作 `Sockets` API 相比，大大减少了复杂度。此外，`Channel` 是很多专有实现的基类，这些实现包括：
* `EmbeddedChannel`
* `LocalServerChannel`
* `NioDatagramChannel`
* `NioSctpChannel`
* `NioSocketChannel`

### EventLoop 接口
`EventLoop` 定义了 Netty 核心抽象：在一个连接周期中处理发生的事件。图3.1 在一个高层次展示了 `Channels`、`EventLoops`、`Threads` 和 `EventLoopGroups` 的关系。关系如下：
* 一个 `EventLoopGroup` 包括了一个或多个 `EventLoop`。
* 一个 `EventLoop` 在生命周期中只会绑定到一个单一的线程 `Thread`。
* 所有被 `EventLoop` 处理的 I/O 事件都会被它绑定的线程 `Thread` 处理。
* 一个 `Channel` 会在一个单一的 `EventLoop` 中注册它的生命周期。
* 一个单一的 `EventLoop` 可能会被分配多个 `Channels`。
 
![图3.1 Channels、EventLoops 和 EventLoopGroups 的关系](/images/00011/01.png "Channels、EventLoops 和 EventLoopGroups 的关系")

值得注意的是：在这个设计中，一个给定 `Channel` 的 I/O 的操作都是由同一个线程执行的，也就是这些操作是单线程的，所以无形中消除了同步的问题。

### ChannelFuture 接口
Netty 中提供了 `ChannelFuture`，这个接口具有 `addListener()`方法，通过这个方法可以注册一个 `ChannelFutureListener`，这个监听器会在方法结束时被调用（无论执行是否成功）。

>__ChannelFuture 的更多介绍__ 
可以把 `ChannelFuture` 看做是存放结果的一个站位空间，这里的结果将会在未来的某个时刻被填满。可以确定的是：对于同一个 `Channel` 的所有操作都会被执行，并且执行顺序与调用的顺序一致。

## ChannelHandler 和 ChannelPipeline
### ChannelHandler 接口
对于应用开发者来说，Netty 中最重要的组件是 `ChannelHandler`。这个组件承载了对输入输出数据的所有应用处理逻辑。

事实上，`ChannelHandler` 几乎可以满足所有的操作，包括：转换数据的格式、处理过程中抛出的异常等。

### ChannelPipeline 接口
`ChannelPipeline` 提供了链式 `ChannelHandlers` 的一个承载容器，并且定义了在这个链中传递输入输出事件的 API。当一个 `Channel` 被创建的时候，它会被自动绑定到它所在的 `ChannelPipeline` 上。

`ChannelHandler` 通过以下步骤安装到 `ChannelPipeline` 上：
* 在 `ServerBootstrap` 上注册一个 `ChannelInitializer` 的实现。
* 当 `ChannelInitializer.initChannel()` 函数被调用的时候，`ChannelInitializer` 将会在 pipeline 中安装一系列的 `ChannelHandler`。
* 然后 `ChannelInitializer` 从 `ChannelPipeline` 中把它自身删除。

图3.2 展示了从 `ChannelHandler` 派生出来的 `ChannelInboundHandler` 和 `ChannelOutboundHandler`。

![图3.2 ChannelHandler 类的继承关系](/images/00011/02.png "图3.2 ChannelHandler 类的继承关系")

图3.3 展示了 Netty 应用中输入和输出端数据流的区别。

![图3.3 输入和输出端数据流的区别](/images/00011/03.png "图3.3 输入和输出端数据流的区别")

>__输入输出处理器的更多介绍__
一个事件可以通过 `ChannelHandlerContext` 传递到链中的下一个处理器，`ChannelHandlerContext` 在每个方法中都会被作为一个输入参数。因为你有时候会忽略不感兴趣的事件，Netty 提供了抽象基础类 `ChannelInboundHandlerAdapter` 和 `ChannelOutboundHandlerAdapter`。每个都提供了方法实现，通过调用 `ChannelHandlerContext` 中的相应方法，可以简化传递到下一个处理者的操作。然后，你可以扩展类并覆盖你感兴趣的方法。

当两种类别处理器在同一个 `ChannelPipeline` 中混合时会发生什么？尽管输入输出 handler 都继承于 `ChannelHandler`，但 Netty 还是会区分 `ChannelInboundHandler` 和 `ChannelOutboundHandler` 的不同实现，并且 __确保数据只在方向相同类型的 handler 之间传递__。

当 `ChannelHandler` 被加入到 `ChannelPipeline` 中时，它会被分配一个 `ChannelHandlerContext`，这个 `ChannelHandlerContext` 代表了 `ChannelHandler` 和 `ChannelPipeline` 之间的绑定关系。

Netty 中发送数据的方法有两种：
* 直接写入到 `Channel`，消息会从 `ChannelPipeline` 的尾部开始传递。
* 写入到 `ChannelHandler` 关联的 `ChannelHandlerContext`，消息会从 `ChannelPipeline` 中的__下一个__handler 开始传递。

### 进一步了解 ChannelHandler
Netty 中有一些适配类，从而减少了常用 `ChannelHandler` 类的工作量。这些适配类提供了相应接口方法的默认实现。

你在写 handle 时经常会用到的适配类包括：
* `ChannelHandlerAdapter`
* `ChannelInboundHandlerAdapter`
* `ChannelOutboundHandlerAdapter`
* `ChannelDuplexHandlerAdapter`

### 编码和解码
当你使用 Netty 发送和接收数据时，需要进行数据格式的转换。输入的消息将会被解码，也就是将输入的字节转换为另一种格式，典型的是转化为一个 Java 对象。如果数据是输出的，相反的操作将会发生：会从现在的格式编码成自己。需要这样做的原因很简单，网络传输的数据是一系列的字节。

所有 Netty 提供的编码解码（encoder/decoder）的适配类都实现于 `ChannelInboundHandler` 或者 `ChannelOutboundHandler`。

你将发现：对于输入数据，`channelRead` 方法或事件会被覆写。对于输入 `Channel` 的每条数据，这个方法都会被调用。然后会调用解码器的 `decode()` 方法，并将解码后的字节传输到 pipeline 中的下一个 `ChannelInboundHandler`。 

而输出数据的模式正好相反：编码器将消息转换为字节，并将这些字节传递到下一个 `ChannelOutboundHandler`。

### 抽象类 SimpleChannelInboundHandler
很常见的是，你的应用需要实现一个 handler 来接受解码的数据并且在数据上实现应用逻辑。创建这样的一个 `ChannelHandler`，你只需要扩展基类 `SimpleChannelInboundHandler<T>`，其中 `T` 是你需要处理的数据对应的 Java 类型。 

这个 handler 中最重要的一个方法是 `channelRead0(ChannelHandlerContext,T)`。

## Bootstrapping
Netty 的启动类可以为应用的网络层提供配置容器。

>__面向连接的协议__
请记住：严格意义上的连接（“connection”）只适用于面向连接的协议，如 TCP。在这样的协议中，可以保证连接的端之间可以按序接收到数据。

相应的，有两种类型的启动类：一个是客户端（`Bootstrap`），另一个是服务端（`ServerBootstrap`）。

下表对这两种类型的启动进行了比较。

类别| Bootstrap| ServerBootstrap |
----|---- |----
网络功能 | 连接远程主机和端口 | 绑定本地端口
`EventLoopGroup` 的数目 | 1 | 2

启动一个客户端只需要一个 `EventLoopGroup`，但是服务端的 `ServerBootstrap` 需要两个（可以是同一个实例），这是为什么？

服务端需要两个区分的 `Channel` 集合。第一个集合包括一个 `ServerChannel`，它代表了服务端自己用于监听（listening）的 socket，绑定到本地端口；第二个集合包括了所有创建的 `Channels`，用于处理输入服务端接收到的各个客户端连接。图3.4 展示了这个模型，并说明了为什么需要2个不同的 `EventLoopGroups`。

![图3.4 服务端的2个 EventLoopGroup](/images/00011/04.png "图3.4 服务端的2个 EventLoopGroup")

与 `ServerChannel` 相关的 `EventLoopGroup` 为 `ServerChannel`分配了一个 `EventLoop`，这个 `EventLoop` 负责为即将到来的连接请求创建 `Channel`。一旦接收到连接，第二个 `EventLoopGroup` 会为创建出来的 `Channel` 分配对应的 `EventLoop`。