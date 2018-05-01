---
title: 《Netty-in-Action》笔记（8）
date: 2018-02-17 00:03:14
tags: [笔记,Netty,Java] 
categories: Netty
---

# Bootstrapping
This chapter covers
* Bootstrapping clients and servers
* Bootstrapping clients from within a `Channel`
* Adding `ChannelHandlers`
* Using `ChannelOptions` and attributes

<!-- more -->

## Bootstrap classes
The bootstrapping class hierarchy consists of an abstract parent class and two concrete bootstrap subclasses, as shown in figure 8.1.
![Figure 8.1 Bootstrapping class hierarchy](/images/00016/01.png "Figure 8.1 Bootstrapping class hierarchy")

>__Why are the bootstrap classes Cloneable?__
>You’ll sometimes need to create multiple channels that have similar or identical settings. To support this pattern without requiring a new bootstrap instance to be created and configured for each channel, `AbstractBootstrap` has been marked `Cloneable`. Calling `clone()` on an already configured bootstrap will return another bootstrap instance that’s immediately usable. 
>
>Note that this creates only a shallow copy of the bootstrap’s `EventLoopGroup`, so the latter will be shared among all of the cloned channels. This is acceptable, as the cloned channels are often short-lived, a typical case being a channel created to make an HTTP request.

The full declaration of `AbstractBootstrap` is
```
public abstract class AbstractBootstrap 
	<B extends AbstractBootstrap<B,C>,C extends Channel>
```

The subclasses are declared as follows:
```
public class Bootstrap
 extends AbstractBootstrap<Bootstrap,Channel>
```
and
```
public class ServerBootstrap
 extends AbstractBootstrap<ServerBootstrap,ServerChannel>
```

## Bootstrapping clients and connectionless protocols
`Bootstrap` is used in clients or in applications that use a connectionless protocol. The following table gives an overview of the class, many of whose methods are inherited from `AbstractBootstrap`.
![](/images/00016/02.png)

### Bootstrapping a client
The `Bootstrap` class is responsible for creating channels for clients and for applications that utilize connectionless protocols, as illustrated in figure 8.2.
![Figure 8.2 Bootstrapping process](/images/00016/03.png "Figure 8.2 Bootstrapping process")

The code in the following listing bootstraps a client that uses the NIO TCP transport.
![](/images/00016/04.png)

### Channel and EventLoopGroup compatibility
The following directory listing is from the package `io.netty.channel`. 
![](/images/00016/05.png)

The following listing shows Incompatible `Channel` and `EventLoopGroup`.
![](/images/00016/06.png)

This code will cause an IllegalStateException because it mixes incompatible transports.

>__More on IllegalStateException__
When bootstrapping, before you call `bind()` or `connect()` you must call the following methods to set up the required components.
* `group()`
* `channel()` or `channnelFactory()`
* `handler()`
>
>Failure to do so will cause an `IllegalStateException`. The `handler()` call is particularly important because it’s needed to configure the `ChannelPipeline`.

## Bootstrapping servers
### The ServerBootstrap class
The following table lists the methods of `ServerBootstrap`.
![](/images/00016/07.png)

### Bootstrapping a server
You may have noticed that table 8.2 lists several methods not present in table 8.1:`childHandler()`, `childAttr()`, and `childOption()`. These calls support operations that are typical of server applications. Specifically, `ServerChannel` implementations are responsible for creating child `Channels`, which represent accepted connections.

Figure 8.3 shows a `ServerBootstrap` creating a `ServerChannel` on `bind()`, and the `ServerChannel` managing a number of child `Channels`.
![Figure 8.3 ServerBootstrap and ServerChannel](/images/00016/08.png "Figure 8.3 ServerBootstrap and ServerChannel")

The code in this listing implements the server bootstrapping shown in figure 8.3.
![](/images/00016/09.png)

## Bootstrapping clients from a Channel
Figure 8.4 shows `EventLoop` shared between channels.
![Figure 8.4 EventLoop shared between channels](/images/00016/10.png "Figure 8.4 EventLoop shared between channels")

Implementing `EventLoop` sharing involves setting the `EventLoop` by calling the `group()` method, as shown in the following listing.
![](/images/00016/11.png)

A general guideline in coding Netty applications: reuse `EventLoops` wherever possible to reduce the cost of thread creation.

## Adding multiple ChannelHandlers during a bootstrap
Netty supplies a special subclass of `ChannelInboundHandlerAdapter`,
```
public abstract class ChannelInitializer<C extends Channel>
 extends ChannelInboundHandlerAdapter
```
which defines the following method:
```
protected abstract void initChannel(C ch) throws Exception;
```

The following listing defines the class `ChannelInitializerImpl` and registers it using the bootstrap’s `childHandler()`. 
![](/images/00016/12.png)

## Using Netty ChannelOptions and attributes
The next listing shows how you can use `ChannelOptions` to configure a `Channel` and an attribute to store an integer value.
![](/images/00016/13.png)

## Bootstrapping DatagramChannels
Netty provides various `DatagramChannel` implementations for this purpose. The only difference is that you don’t call `connect()` but only `bind()`, as shown next.
![](/images/00016/14.png)

## Shutdown
Above all, you need to shut down the `EventLoopGroup`, which will handle any pending events and tasks and subsequently release all active threads.

The following listing meets the definition of a graceful shutdown.
![](/images/00016/15.png)

Alternatively, you can call `Channel.close()` explicitly on all active channels before calling `EventLoopGroup.shutdownGracefully()`. But in all cases, remember to shut down the `EventLoopGroup` itself.



