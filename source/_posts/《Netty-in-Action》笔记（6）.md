---
title: 《Netty-in-Action》笔记（6）
date: 2018-02-14 15:15:25
tags: [笔记,Netty,Java] 
categories: Netty
---

# ChannelHandler and ChannelPipeline
This chapter covers
* The `ChannelHandler` and `ChannelPipeline` APIs
* Detecting resource leaks
* Exception handling

<!-- more -->

## The ChannelHandler family
### The Channel lifecycle
`Interface Channel` defines a simple but powerful state model that’s closely related to the `ChannelInboundHandler` API. The four `Channel` states are listed in the following table.
![](/images/00014/01.png)

The normal lifecycle of a `Channel` is shown in figure 6.1.
 ![Figure 6.1 Channel state model](/images/00014/02.png "Figure 6.1 Channel state model")

### The ChannelHandler lifecycle
The lifecycle operations defined by `interface ChannelHandler`, listed in the following table, are called after a `ChannelHandler` has been added to, or removed from, a `ChannelPipeline`.
![](/images/00014/03.png)

Netty defines the following two important subinterfaces of `ChannelHandler`:
* `ChannelInboundHandler` — Processes inbound data and state changes of all kinds
* `ChannelOutboundHandler` — Processes outbound data and allows interception of all operations 

### Interface ChannelInboundHandler
The following table lists the lifecycle methods of `interface ChannelInboundHandler`. These are called when data is received or when the state of the associated `Channel` changes.
![](/images/00014/04.png)

When a `ChannelInboundHandler` implementation overrides `channelRead()`, it is responsible for explicitly releasing the memory associated with pooled `ByteBuf` instances. Netty provides a utility method for this purpose, `ReferenceCountUtil.release()`, as shown next.
![](/images/00014/05.png)

A simpler alternative is to use `SimpleChannelInboundHandler`.
![](/images/00014/06.png) 

Because `SimpleChannelInboundHandler` releases resources automatically, __you shouldn’t store references to any messages for later use__, as these will become invalid.

### Interface ChannelOutboundHandler
The following table shows all of the methods defined locally by `ChannelOutboundHandler`(leaving out those inherited from `ChannelHandler`).
![](/images/00014/07.png) 

>__CHANNELPROMISE VS. CHANNELFUTURE__
Most of the methods in `ChannelOutboundHandler` take a `ChannelPromise` argument to be notified when the operation completes.  `ChannelPromise` is a subinterface of `ChannelFuture` that defines the writable methods, such as `setSuccess()` or `setFailure()`, thus making `ChannelFuture` immutable.

### ChannelHandler adapters
The resulting class hierarchy is shown in figure 6.2.
![Figure 6.2 ChannelHandlerAdapter class hierarchy](/images/00014/08.png "Figure 6.2 ChannelHandlerAdapter class hierarchy") 

The method bodies provided in `ChannelInboundHandlerAdapter` and `ChannelOutboundHandlerAdapter` call the equivalent methods on the associated `ChannelHandlerContext`, thereby forwarding events to the next `ChannelHandler` in the pipeline. 

### Resource management
Netty provides class `ResourceLeakDetector`, which will sample about 1% of your application’s buffer allocations to check for memory leaks. If a leak is detected, a log message similar to the following will be produced:

```
LEAK: ByteBuf.release() was not called before it's garbage-collected. Enable 
advanced leak reporting to find out where the leak occurred. To enable 
advanced leak reporting, specify the JVM option
'-Dio.netty.leakDetectionLevel=ADVANCED' or call 
ResourceLeakDetector.setLevel().
```

Netty currently defines the four leak detection levels, as listed in the following table.
![](/images/00014/09.png)

The leak-detection level is defined by setting the following Java system property to one of the values in the table:
```
java -Dio.netty.leakDetectionLevel=ADVANCED
```

This listing shows how to release the message.
![](/images/00014/10.png)

On the outbound side, if you handle a `write()` operation and discard a message, you’re responsible for releasing it. The next listing shows an implementation that discards all written data.
![](/images/00014/11.png)

It’s important not only to release resources but also to notify the `ChannelPromise`. Otherwise a situation might arise where a `ChannelFutureListener` has not been notified about a message that has been handled.

In sum, it is the responsibility of the user to call `ReferenceCountUtil.release()` if a message is consumed or discarded and not passed to the next `ChannelOutboundHandler` in the `ChannelPipeline`. If the message reaches the actual transport layer, it will be released automatically when it’s written or the `Channel` is closed.

## Interface ChannelPipeline
Every new `Channel` that’s created is assigned a new `ChannelPipeline`. This association is permanent.

>__ChannelHandlerContext__
A `ChannelHandlerContext` enables a `ChannelHandler` to interact with its `ChannelPipeline` and with other handlers. A handler can notify the next `ChannelHandler` in the `ChannelPipeline` and even dynamically modify the `ChannelPipeline` it belongs to.

### Modifying a ChannelPipeline
A `ChannelHandler` can modify the layout of a `ChannelPipeline` in real time by adding, removing, or replacing other `ChannelHandlers`. (It can remove itself from the `ChannelPipeline` as well.) The relevant methods are listed in the following table.
![](/images/00014/12.png)

This listing shows these methods in use.
![](/images/00014/13.png)

>__ChannelHandler execution and blocking__
Normally each `ChannelHandler` in the `ChannelPipeline` processes events that are passed to it by its `EventLoop` (the I/O thread). It’s critically important not to block this thread as it would have a negative effect on the overall handling of I/O.

The following table shows the `ChannelPipeline` operations for accessing `ChannelHandlers`.
![](/images/00014/14.png)

### Firing events
The following table lists the inbound operations, which notify `ChannelInboundHandlers` of events occurring in the `ChannelPipeline`.
![](/images/00014/15.png)

The following table lists the outbound operations of the `ChannelPipeline` API.
![](/images/00014/16.png)

In summary,
* A `ChannelPipeline` holds the `ChannelHandlers` associated with a `Channel`.
* A `ChannelPipeline` can be modified dynamically by adding and removing `ChannelHandlers` as needed.
* `ChannelPipeline` has a rich API for invoking actions in response to inbound and outbound events.

## Interface ChannelHandlerContext
A `ChannelHandlerContext` represents an association between a `ChannelHandler` and a `ChannelPipeline` and is created whenever a `ChannelHandler` is added to a `ChannelPipeline`. 

The methods called on a `ChannelHandlerContext` will start at the current associated `ChannelHandler` and propagate only to the next `ChannelHandler` in the pipeline that is capable of handling the event.

The following table summarizes the `ChannelHandlerContext` API.
![](/images/00014/17.png)

When using the `ChannelHandlerContext` API, please keep the following points in mind:
* The `ChannelHandlerContext` associated with a `ChannelHandler` never changes, so it’s safe to cache a reference to it.
* `ChannelHandlerContext` methods involve a shorter event flow than do the identically named methods available on other classes. This should be exploited where possible to provide maximum performance.

### Using ChannelHandlerContext
Figure 6.4 shows the relationships.
![Figure 6.4 The relationships among Channel, ChannelPipeline, ChannelHandler, and ChannelHandlerContext](/images/00014/18.png "Figure 6.4 The relationships among Channel, ChannelPipeline, ChannelHandler, and ChannelHandlerContext")

In the following listing you acquire a reference to the Channel from a `ChannelHandlerContext`. Calling `write()` on the `Channel` causes a write event to flow all the way through the pipeline.
![](/images/00014/19.png)

The next listing shows a similar example, but writing this time to a `ChannelPipeline`.
![](/images/00014/20.png)

As you can see in figure 6.5, the flows in listings 6.6 and 6.7 are identical. It’s important to note that although the `write()` invoked on either the `Channel` or the `ChannelPipeline` operation propagates the event all the way through the pipeline, the movement from one handler to the next at the `ChannelHandler` level is invoked on the `ChannelHandlerContext`.
![](/images/00014/21.png "Figure 6.5 Event propagation via the Channel or the ChannelPipeline")

If you want to propagate an event starting at a specific point in the `ChannelPipeline`, the following listing and figure 6.6 illustrate this use.
![](/images/00014/22.png)

As shown in figure 6.6, the message flows through the `ChannelPipeline` starting at the _next_ `ChannelHandler`, bypassing all the preceding ones.
![Figure 6.6 Event flow for operations triggered via the ChannelHandlerContext](/images/00014/23.png "Figure 6.6 Event flow for operations triggered via the ChannelHandlerContext")

###  Advanced uses of ChannelHandler and ChannelHandlerContext
The following list shows caching a `ChannelHandlerContext`.
![](/images/00014/24.png)

Because a `ChannelHandler` can belong to more than one `ChannelPipeline`, it can be bound to multiple `ChannelHandlerContext` instances. A `ChannelHandler` intended for this use must be annotated with `@Sharable`; otherwise, attempting to add it to more than one `ChannelPipeline` will trigger an exception. This listing shows a correct implementation of this pattern.
![](/images/00014/25.png)

Conversely, the code in listing 6.11 will cause problems.
![](/images/00014/26.png)

In summary, use `@Sharable` only if you’re certain that your `ChannelHandler` is thread-safe.

## Exception handling
### Handling inbound exceptions
The following listing shows a simple example that closes the `Channel` and prints the exception’s stack trace.
![](/images/00014/27.png)

To summarize,
* The default implementation of `ChannelHandler.exceptionCaught()` forwards the current exception to the next handler in the pipeline.
* If an exception reaches the end of the pipeline, it’s logged as unhandled.
* To define custom handling, you override `exceptionCaught()`. It’s then your decision whether to propagate the exception beyond that point.

### Handling outbound exceptions
Notification mechanisms:
* Every outbound operation returns a `ChannelFuture`. The `ChannelFutureListeners` registered with a `ChannelFuture` are notified of success or error when the operation completes.
* Almost all methods of `ChannelOutboundHandler` are passed an instance of `ChannelPromise`. As a subclass of `ChannelFuture`, `ChannelPromise` can also be assigned listeners for asynchronous notification. But `ChannelPromise` also has writable methods that provide for immediate notification:
```
ChannelPromise setSuccess();
ChannelPromise setFailure(Throwable cause);
```

The following listing uses this approach to add a `ChannelFutureListener` that will print the stack trace and then close the `Channel`.
![](/images/00014/28.png)

The code shown next will have the same effect as the previous listing.
![](/images/00014/29.png)

