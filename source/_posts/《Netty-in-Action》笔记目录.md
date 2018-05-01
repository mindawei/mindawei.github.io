---
title: 《Netty-in-Action》笔记目录
date: 2018-03-01 20:51:30
tags: [笔记,Netty,Java] 
categories: Netty
---

对之前摘录的一个目录。

<!-- more -->

# [01 Netty — 异步和事件驱动](https://mindawei.github.io/2018/02/08/01%20Netty%20%E2%80%94%20%E5%BC%82%E6%AD%A5%E5%92%8C%E4%BA%8B%E4%BB%B6%E9%A9%B1%E5%8A%A8/)
* Java 中的网络使用
* 介绍 Netty
* Netty 的核心组件

# [02 你的第一个Netty应用](https://mindawei.github.io/2018/02/09/02%20%E4%BD%A0%E7%9A%84%E7%AC%AC%E4%B8%80%E4%B8%AANetty%E5%BA%94%E7%94%A8/)
* 建立开发环境
* 写一个回写应用的服务器和客户端
* 构建和测试程序

# [03 Netty的组件和设计](https://mindawei.github.io/2018/02/10/03%20Netty%E7%9A%84%E7%BB%84%E4%BB%B6%E5%92%8C%E8%AE%BE%E8%AE%A1/)
* Netty 技术和架构方面的介绍
* `Channel`、`EventLoop` 和 `ChannelFuture`
* `ChannelHandler` 和 `ChannelPipeline`
* 启动（ Bootstrapping）

# [04 传输](https://mindawei.github.io/2018/02/11/04%20%E4%BC%A0%E8%BE%93/)
* OIO：阻塞传输
* NIO：异步传输
* 本地传输：和 JVM 异步交互
* 测试你的 `ChannelHandler`

# [05 ByteBuf](https://mindawei.github.io/2018/02/12/05%20ByteBuf/)
* `ByteBuf`：Netty 的数据容器
* API 细节
* 用例
* 内存分配

# [06 ChannelHandler and ChannelPipeline](https://mindawei.github.io/2018/02/14/%E3%80%8ANetty-in-Action%E3%80%8B%E7%AC%94%E8%AE%B0%EF%BC%886%EF%BC%89/)
This chapter covers
* The `ChannelHandler` and `ChannelPipeline` APIs
* Detecting resource leaks
* Exception handling

# [07 EventLoop and threading model](https://mindawei.github.io/2018/02/16/%E3%80%8ANetty-in-Action%E3%80%8B%E7%AC%94%E8%AE%B0%EF%BC%887%EF%BC%89/)
This chapter covers
* Threading model overview
* Event loop concept and implementation
* Task scheduling
* Implementation details

# [08 Bootstrapping](https://mindawei.github.io/2018/02/17/%E3%80%8ANetty-in-Action%E3%80%8B%E7%AC%94%E8%AE%B0%EF%BC%888%EF%BC%89/)
This chapter covers
* Bootstrapping clients and servers
* Bootstrapping clients from within a `Channel`
* Adding `ChannelHandlers`
* Using `ChannelOptions` and attributes

# [09 Unit testing](https://mindawei.github.io/2018/02/18/%E3%80%8ANetty-in-Action%E3%80%8B%E7%AC%94%E8%AE%B0%EF%BC%889%EF%BC%89/)
This chapter covers
* Unit testing
* Overview of `EmbeddedChannel`
* Testing `ChannelHandlers` with `EmbeddedChannel`

# [10 The codec framework](https://mindawei.github.io/2018/02/18/%E3%80%8ANetty-in-Action%E3%80%8B%E7%AC%94%E8%AE%B0%EF%BC%8810%EF%BC%89/)
This chapter covers
* An overview of decoders, encoders and codecs
* Netty’s codec classes

# [11 Provided ChannelHandlers and codecs](https://mindawei.github.io/2018/02/19/%E3%80%8ANetty-in-Action%E3%80%8B%E7%AC%94%E8%AE%B0%EF%BC%8811%EF%BC%89/)
This chapter covers
* Securing Netty applications with SSL/TLS
* Building Netty HTTP/HTTPS applications
* Handling idle connections and timeouts
* Decoding delimited and length-based protocols
* Writing big data

# [12 WebSocket](https://mindawei.github.io/2018/02/22/%E3%80%8ANetty-in-Action%E3%80%8B%E7%AC%94%E8%AE%B0%EF%BC%8812%EF%BC%89/)
This chapter covers
* The concept of a real-time web
* The WebSocket protocol
* Building a WebSocket-based chat room server with Netty


# [13 Broadcasting events with UDP](https://mindawei.github.io/2018/02/22/%E3%80%8ANetty-in-Action%E3%80%8B%E7%AC%94%E8%AE%B0%EF%BC%8813%EF%BC%89/)
This chapter covers
* An overview of UDP
* A sample broadcasting application
