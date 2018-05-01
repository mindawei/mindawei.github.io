---
title: 02 你的第一个Netty应用
date: 2018-02-09 13:13:14
tags: [笔记,Netty,Java] 
categories: Netty
---
点击查看 [《Netty in Action》笔记目录](https://mindawei.github.io/2018/03/01/%E3%80%8ANetty-in-Action%E3%80%8B%E7%AC%94%E8%AE%B0%E7%9B%AE%E5%BD%95/)。

本文是对[《Netty in Action》](https://download.csdn.net/download/u010738033/10245651)第2章内容的笔记和翻译，主要内容包括：
* 建立开发环境
* 写一个回写应用的服务器和客户端
* 构建和测试程序

<!-- more -->

## 搭建开发环境
1. 下载和安装 JDK
2. 下载和安装 IDE
3. 下载和安装 Apache Maven
4. 配置工具集

## Netty 客户端和服务器概览

图2.1在一个较高的层次展示了将要写的回写客户端和服务端程序。
![图2.1 回写程序的客户端和服务端](/images/00010/01.png "图2.1 回写程序的客户端和服务端")

## Echo 程序服务端
所有的 Netty 应用服务端都需要以下要素：
* _至少一个_ `ChannelHandler`：这个组件实现了服务端对接收到的客户端数据的处理，是业务的处理逻辑。
* _Bootstrapping_ ：这是配置服务端的启动的代码。它至少会绑定服务端的某个端口，从而监听连接的请求。

### ChannelHandlers 和业务逻辑
你需要实现 `ChannelInboundHandler` 接口。这个接口定义了输入事件的响应行为方法。这个简单的应用程序只需要实现这个接口的较少方法，所以继承 `ChannelInboundHandlerAdapter` 就足够了，这个类可以提供 `ChannelInboundHandler` 的一个默认实现。

我们将会对以下这些方法感兴趣：
* `channelRead()` ：每个数据输入的时候都会调用。
* `channelReadComplete()` ：通知 handler 最近一次 `channelRead()` 调用处理的是最后一条数据。
* `exceptionCaught()` ：在读数据过程如果抛出异常，则会调用这个方法。

![](/images/00010/02.png)

>__当捕获异常时会发生什么?__
每个 `Channel` 都会有一个关联的 `ChannelPipeline`（它持有了一个链式的 `ChannelHandler` 实例）。默认情况下, 一个 handler 会将方法调用传递给链中的下一个 hander。因此，如果 `exceptionCaught()` 在链中没有被实现，那么捕捉的异常会传递到 `ChannelPipeline` 的末端，并会被日志记录下来。基于这个原因，你的应用至少要提供一个实现了 `exceptionCaught()` 的 `ChannelHandler`。

### 启动服务端
服务的启动包括以下几个方面：
* 绑定服务端将要监听的端口，并且接受将要到来的连接请求。
* 配置 `Channels` 来通知 `EchoServerHandler` 实例即将到来的输入数据。

![](/images/00010/03.png)

代码的主要组成部分包括：
* `EchoServerHandler` 实现了基本的业务逻辑。
* `main()` 函数负责启动服务器。

在启动过程中需要经历以下步骤：
* 创建一个 `ServerBootstrap` 实例来启动和绑定服务。
* 创建一个 `NioEventLoopGroup` 实例来进行事件处理，这些事件包括：接受新的连接、读写数据等。
* 确定本地需要绑定的 `InetSocketAddress`。
* 为每个新的 `Channel` 分配一个 `EchoServerHandler` 实例。
* 调用 `ServerBootstrap.bind()` 来绑定服务。

## Echo 程序客户端
Echo 客户端将会做以下这些事情：
1. 连接服务端。
2. 发送一条或多条消息。
3. 对于每条消息，等待服务器端发回同样的消息。
4. 关闭连接。

### 通过 ChannelHandlers 实现客户端逻辑
和服务端相同，客户端将会有 `ChannelInboundHandler` 来处理数据。在这个类中，你将要扩展 `SimpleChannelInboundHandler` 来处理所有需要的任务，具体如清单2.3所示。这需要覆盖以下的这些方法：
* `channelActive()`：当与服务端的连接建立后会被调用。
* `channelRead0()`：当从服务端收到数据后会被调用。
* `exceptionCaught()`：当处理过程中发生异常时会被调用。

![](/images/00010/04.png)

需要注意的是，服务器端发送过来的数据可能被分成多个小块。也就是说，如果服务端发送5个字节，那么并不能保证所有的5个字节都被一次收到。

> __SimpleChannelInboundHandler 对比 ChannelInboundHandler__
两个方面的考量：业务逻辑是怎么样的、Netty 是如何管理资源的。<br>
在客户端，当 `channelRead0()` 完成的时候, 你将接收并处理完毕输入的数据。当这个方法返回的时候，`SimpleChannelInboundHandler` 帮助我们释放用于存储消息的 `ByteBuf` 内存引用。<br>
在 `EchoServerHandler` 中，你还需要将接收到的数据回显给发送端， 并且异步的 `write()` 在 `channelRead()` 返回的时候还不一定可以完成（如列表 2.1 所示）。因此，`EchoServerHandler` 扩展了 `ChannelInboundHandlerAdapter`, 后者并没有在方法返回的时候释放消息所占用的资源。<br>
当 EchoServerHandler 类的 `channelReadComplete()` 方法中的 `writeAndFlush()` 被调用后，消息将会被释放（如列表2.1所示）。<br>

### 启动客户端

![](/images/00010/05.png)

你可以在客户端和服务端之间使用不同的传输传输协议。例如，在服务端使用 NIO，在客户端使用 OIO。 

客户端和服务端创建的要点总结如下：
* 创建 `Bootstrap` 实例来初始化客户端。
* 创建 `NioEventLoopGroup` 实例来进行事件处理，这些事件包括：接受新的连接、读写数据等。
* 创建 `InetSocketAddress` 来帮助服务端建立来连接。
* 当连接建立的时候，在流水线中安装 `EchoClientHandler`。
* 在所有的事情建立完毕后，将会调用 `Bootstrap.connect()` 来连接远端。