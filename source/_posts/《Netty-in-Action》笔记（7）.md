---
title: 《Netty-in-Action》笔记（7）
date: 2018-02-16 14:29:55
tags: [笔记,Netty,Java] 
categories: Netty
---

# EventLoop and threading model
This chapter covers
* Threading model overview
* Event loop concept and implementation
* Task scheduling
* Implementation details

<!-- more -->

## Threading model overview
The basic thread pooling pattern can be described as:
* A Thread is selected from the pool’s free list and assigned to run a submitted task (an implementation of `Runnable`).
* When the task is complete, the `Thread` is returned to the list and becomes available for reuse.

This pattern is illustrated in figure 7.1.
![Figure 7.1 Executor execution logic](/images/00015/01.png "Figure 7.1 Executor execution logic")

## Interface EventLoop
The corresponding programming construct is often referred to as an _event loop_, a term Netty adopts with `interface io.netty.channel.EventLoop`. 

The basic idea of an event loop is illustrated in the following listing, where each task is an instance of `Runnable` (as in figure 7.1).
![](/images/00015/02.png)

Netty’s `EventLoop` is part of a collaborative design that employs two fundamental APIs: concurrency and networking. First, the package `io.netty.util.concurrent` builds on the JDK package `java.util.concurrent` to provide thread executors. Second, the classes in the package `io.netty.channel` extend these in order to interface with `Channel` events. The resulting class hierarchy is seen in figure 7.2.
![Figure 7.2 EventLoop class hierarchy](/images/00015/03.png "Figure 7.2 EventLoop class hierarchy")

In this model, an `EventLoop` is powered by exactly one `Thread` that never changes, and tasks (`Runnable` or `Callable`) can be submitted directly to `EventLoop` implementations for immediate or scheduled execution.

Note that Netty’s `EventLoop`, while it extends `ScheduledExecutorService`, defines only one method, `parent()`. This method is intended to return a reference to the `EventLoopGroup` to which the current `EventLoop` implementation instance belongs.
```
public interface EventLoop extends EventExecutor, EventLoopGroup {
 @Override
 EventLoopGroup parent();
}
```

>__EVENT/TASK EXECUTION ORDER__ Events and tasks are executed in FIFO (first-in-first-out) order. 

### I/O and event handling in Netty 4
Event-handling logic must be generic and flexible enough to handle all possible use cases. Therefore, in Netty 4 _all I/O operations and events are handled by the `Thread` that has been assigned to the `EventLoop`_.

### I/O operations in Netty 3
The threading model used in previous releases guaranteed only that inbound (previously called upstream) events would be executed in the so-called I/O thread (corresponding to Netty 4’s `EventLoop`). All outbound (downstream) events were handled by the calling thread, which might be the I/O thread or any other. 

## Task scheduling
### JDK scheduling API
Before Java 5, task scheduling was built on `java.util.Timer`, which uses a background `Thread` and has the same limitations as standard threads. Subsequently, the JDK provided the package `java.util.concurrent`, which defines the `interface ScheduledExecutorService`. Table 7.1 shows the relevant factory methods of `java.util.concurrent.Executors`.
![Table 7.1 The java.util.concurrent.Executors factory methods](/images/00015/04.png "Table 7.1 The java.util.concurrent.Executors factory methods ")

The next listing shows how to use `ScheduledExecutorService` to run a task after a 60-second delay.
![](/images/00015/05.png)

### Scheduling tasks using EventLoop
The `ScheduledExecutorService` implementation has limitations, such as the fact that extra threads are created as part of pool management. 
![](/images/00015/06.png)

To schedule a task to be executed every 60 seconds, use `scheduleAtFixedRate()`, as shown next.
![](/images/00015/07.png)

As we noted earlier, Netty’s `EventLoop` extends `ScheduledExecutorService` (see figure 7.2), so it provides all of the methods available with the JDK implementation, including `schedule()` and `scheduleAtFixedRate()`, used in the preceding examples.

To cancel or check the state of an execution, use the `ScheduledFuture` that’s returned for every asynchronous operation. This listing shows a simple cancellation operation.
![](/images/00015/08.png)

## Implementation details
### Thread management
Figure 7.3 shows the execution logic used by EventLoop to schedule tasks.
![Figure 7.3 EventLoop execution logic](/images/00015/09.png "Figure 7.3 EventLoop execution logic")

We stated earlier the importance of not blocking the current I/O thread. We’ll say it again in another way: “Never put a long-running task in the execution queue, because it will block any other task from executing on the same thread.” If you must make blocking calls or execute long-running tasks, we advise the use of a dedicated EventExecutor. (See the sidebar “ChannelHandler execution and blocking” in section 6.2.1.)

### EventLoop/thread allocation
Figure 7.4 displays an `EventLoopGroup` with a fixed size of three `EventLoops` (each powered by one `Thread`). The `EventLoops` (and their `Threads`) are allocated directly when the `EventLoopGroup` is created to ensure that they will be available when needed.
![Figure 7.4 EventLoop allocation for non-blocking transports (such as NIO and AIO)](/images/00015/10.png "Figure 7.4 EventLoop allocation for non-blocking transports (such as NIO and AIO)")

__BLOCKING TRANSPORTS__
The design for other transports such as OIO (old blocking I/O) is a bit different, as illustrated in figure 7.5.
![Figure 7.5 EventLoop allocation of blocking transports (such as OIO)](/images/00015/11.png "Figure 7.5 EventLoop allocation of blocking transports (such as OIO)")
