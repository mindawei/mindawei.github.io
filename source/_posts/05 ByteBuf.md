---
title: 05 ByteBuf
date: 2018-02-12 10:25:07
tags: [笔记,Netty,Java] 
categories: Netty
---

点击查看 [《Netty in Action》笔记目录](https://mindawei.github.io/2018/03/01/%E3%80%8ANetty-in-Action%E3%80%8B%E7%AC%94%E8%AE%B0%E7%9B%AE%E5%BD%95/)。

本文是对[《Netty in Action》](https://download.csdn.net/download/u010738033/10245651)第5章内容的笔记和翻译，主要内容包括：
* `ByteBuf`：Netty 的数据容器
* API 细节
* 用例
* 内存分配

<!-- more -->

## ByteBuf API

Netty 数据处理的 API 暴露在两个组件上：抽象类 `ByteBuf` 和接口 `ByteBufHolder`。

`ByteBuf` API 的一些优点如下：
* 对于用户定义的缓冲区类型也是可扩展的。 
* 通过内置的复合缓冲类型实现了透明的零拷贝。
* 容量会随着需求扩充（类似 JDK 中的 `StringBuilder`）。
* 在读写模式中切换不需要调用 `ByteBuffer` 的 ` flip()` 方法。
* 读写使用不同的指示下标。
* 支持方法链接。
* 支持引用计数。
* 支持池化。

## ByteBuf 类：Netty 的数据容器
### 它是如何工作的
* `ByteBuf` 维护了两个不同的下标指示：一个用于读，一个用于写。
* `ByteBuf` 以 `read` 或者 `write` 开头的方法会将读下标往前推进，而 `set` 和 `get` 开头的方法__不会__。
* `ByteBuf` 的最大容量是可以被确定的，如果写的数据超过界限，那么会触发一个异常（默认的容量限制是 `Integer.MAX_VALUE`）。

图5.1 展示了 `ByteBuf` 为空时候的状态。
![图5.1 一个下标指向0，容量为16字节的ByteBuf](/images/00013/01.png "图5.1 一个下标指向0，容量为16字节的ByteBuf")

### ByteBuf 的使用方法
#### Heap Buffer
`ByteBuf` 最常用的使用方式是将数据存储到 JVM 的堆空间中去。通过指向一个 _backing array_, 这个模式可以在不使用池化技术的时候，快速地分配和回收内存。
![](/images/00013/02.png)

__注意：__ 当 `hasArray()` 返回 `false` 时，尝试访问 `backing array` 会触发 `UnsupportedOperationException` 异常。这和适用 JDK 中的 `ByteBuffer` 类似。

#### Direct Buffer
_Direct buffer_ 是 `ByteBuf` 的另一种使用方式。JDK 1.4 版本中和 NIO 一起引入的 `ByteBuffer` 类可以允许 JVM 通过本地调用分配内存。这样在调用本地 I/O 操作时，可以避免在数据缓冲区和中间缓冲区之间拷贝数据内容。

`ByteBuffer` 的 Javadoc 明确地指出：direct buffer 的数据内容不在堆垃圾回收的管理范围内。这解释了为什么 direct buffer 适用于网络数据传输。如果你的数据存储在基于堆分配的缓冲区中，那么在发送数据到 socket 前，JVM 会在内部将你缓冲区的数据拷贝到 direct buffer 中去。

Direct buffer 主要的缺点是：它们的分配和释放的开销比 heap-based buffer 更大一点。 
![](/images/00013/03.png)

#### 混合 Buffer
_Composite buffer_ 集成了多个 `ByteBuf`。你可以按需添加和删除 `ByteBuf` 实例，这个特性是 JDK 中的 `ByteBuffer` 实现所不具备的。

__警告：__`CompositeByteBuf` 中的 `ByteBuf` 实例可能包括 direct 和 nondirect 分配。如果只有一个实例，那么在 `CompositeByteBuf` 上调用 `hasArray()` 会返回这个实例的 `hasArray()` 值；否则会返回 `false`。

`CompositeByteBuf` 在暴露通用 `ByteBuf` API 的时候，消除了不必要的拷贝。图5.2 展示了它的结构。

![图5.2 持有 header 和 body 的 CompositeByteBuf](/images/00013/04.png "图5.2 持有 header 和 body 的 CompositeByteBuf")

下面的清单展示了JDK 的 `ByteBuffer` 是如何实现这样的需求的。
![](/images/00013/05.png)

下面的清单展示了使用 `CompositeByteBuf` 的版本。
![](/images/00013/06.png)

`CompositeByteBuf` 可能不允许访问 backing array，所以 `CompositeByteBuf` 的数据访问和 direct buffer 的模式类似，如下所示。
![](/images/00013/07.png)

Netty 通过使用 `CompositeByteBuf` 优化了 socket I/O 操作，消除了 JDK buffer 实现的性能和内存缺陷。

## Byte 级别的操作
### 随机访问
下面的清单展示了对存储机制的封装，可以非常方便地通过迭代访问 `ByteBuf` 的内容。
![](/images/00013/08.png)

值得注意的是，通过输入的 index 参数访问数据，不会修改 `readerIndex` 或 `writerIndex` 的值。这些下标可以通过调用 `readerIndex(index)` 或 `writerIndex(index)` 手工地推进。

### 顺序访问
`ByteBuf` 同时有读和写两个指示下标，而 JDK 的 `ByteBuffer` 只有一个，因此在 JDK 中需要调用 `flip()` 来切换读写模式。图5.3 展示了一个 `ByteBuf` 被两个下标分为了三个部分。
![图5.3 ByteBuf 内部分段](/images/00013/09.png "图5.3 ByteBuf 内部分段")

### 可丢弃字节
图5.4 展示了在图5.3 中展示的 buffer 上调用 `discardReadBytes()` 后的结果。你可以看到 discardable bytes 中的空间可以被写操作所用了。值得注意的是，在 `discardReadBytes()` 方法调用后，writable 端中的内容并不保证不丢失。
![图5.4 丢弃读过的数据后的 ByteBuf](/images/00013/10.png "图5.4 丢弃读过的数据后的 ByteBuf")

你可能倾向于频繁地调用 `discardReadBytes()` 来最大化可写的空间，但是请注意这很有可能导致内存拷贝，因为可读的字节需要被移动到 buffer 开始的地方。
### 可读字节
下面清单展示了如何读取可读的字节。
![](/images/00013/11.png)

### 可写字节
下面清单展示了如何用随机整数填满 buffer，直到空间用完。
![](/images/00013/12.png)

### 下标管理
你可以通过 `markReaderIndex()`、`markWriterIndex()`、`resetReaderIndex()` 和 `resetWriterIndex()` 这些函数来设置和重置 `ByteBuf` 的读写下标。 

你也可以通过调用 `readerIndex(int)` 或 `writerIndex(int)` 函数来移动下标。

图5.6 展示了 `ByteBuf` 在 `clear()` 被调用后的结果（调用前是图5.3）。
![图5.6 clear()调用结果](/images/00013/13.png "图5.6 clear()调用结果")

### 搜索操作
有多个方法可以确定 `ByteBuf` 中需要搜索的值的位置。最简单的一个方法是 `indexOf()`。

下面的清单展示了如何搜索 `\r`。
![](/images/00013/14.png)

### 派生 buffer
一个派生 buffer 提供了 `ByteBuf` 持有内容的特定格式视图。这些视图可以通过下面的方法创建：
* `duplicate()`
* `slice()`
* `slice(int, int)`
* `Unpooled.unmodifiableBuffer(…)`
* `order(ByteOrder)`
* `readSlice(int)`

每个方法都返回了一个自带读、写、标记下标的 `ByteBuf` 实例。内部的存储是共享的，这种模式和 JDK 的 `ByteBuffer` 是一样的。这使得派生出来的 buffer 的创建代价不是很大，但这也意味着：__如果你修改了内容，那么原来实例的数据内容也会改变__，所以要谨慎。

> __Bytebuf 拷贝__ 
> 如果你需要对 buffer 进行真实的拷贝，那可以使用 `copy()` 或者 `copy(int,int)`。和派生的 buffer 不同，`ByteBuf` 的这个调用会返回一个独立的数据拷贝。

下面的清单展示了如何处理 `slice (int, int)` 方法产生的 `ByteBuf` 段。
![](/images/00013/15.png)

接下来让我们看看 `ByteBuf` 的 copy 和 slice 的不同。
![](/images/00013/16.png)

### 读写操作
下表展示了常用的 `get()` 方法。 
![](/images/00013/17.png)

下表展示了常用的 `set()` 方法。 
![](/images/00013/18.png)

下面的清单展示了 `get()` 和 `set()` 方法的使用，可以看到它们没有改变读写下标。
![](/images/00013/19.png)

下表展示了常用的 `read()` 方法
![](/images/00013/20.png)

下表展示了常用的 `write()` 方法
![](/images/00013/21.png)

下面的清单展示了这些方法的使用。
![](/images/00013/22.png)

### 更多操作
下表展示了 `ByteBuf` 其它一些常用的方法。
![](/images/00013/23.png)

## ByteBufHolder 接口
`ByteBufHolder` 有一些方法可以访问底层数据和引用计数。下表展示了这些方法（除去那些继承于 `ReferenceCounted` 的方法）。
![](/images/00013/24.png)

如果你想将一个消息对象的实体数据存储在 `ByteBuf` 中，那么 `ByteBufHolder` 将是一个不错的选择。

## ByteBuf 分配
### 按需：ByteBufAllocator 接口
为了减少内存分配和回收的开销，Netty 提供了采用池化技术的 `ByteBufAllocator`。

下表展示了 `ByteBufAllocator` 提供的操作。
![](/images/00013/25.png)

你可以通过 `Channel`（每个 `Channel` 都有一个不同的实例）或者与 `ChannelHandler` 绑定的 `ChannelHandlerContext` 来获得 `ByteBufAllocator` 的引用。
![](/images/00013/26.png)

Netty 提供了2种 ByteBufAllocator 实现：`PooledByteBufAllocator` 和 `UnpooledByteBufAllocator`。前者会池化 `ByteBuf` 实例来提升性能和最小化内存分配。这个实现采用了一个高效的内存分配方法 __jemalloc__，这种方法当前被多个操作系统所采用。 

### 没有池化的 buffer
Netty 提供了 `Unpooled` 工具类，该工具类提供了静态辅助函数来创建不池化的 `ByteBuf` 实例。下表展示了这个类的一些重要方法。
![](/images/00013/27.png)

### ByteBufUtil
`ByteBufUtil` 提供了静态辅助函数来帮助操控 `ByteBuf`。
* `hexdump()`
* `boolean equals(ByteBuf, ByteBuf)`

## 引用计数
引用计数是一个用于优化内存使用和提高性能的技术：当对象没有其他对象引用它的时候，对它进行释放。Netty 在版本4中为 `ByteBuf` 和 `ByteBufHolder` 引入了引用计数，这两者都实现了 `ReferenceCounted` 接口。 

引用计数对于池化实现（如 `PooledByteBufAllocator`）是非常重要的，这减少了内存分配的开销。下面清单展示了使用例子。
![](/images/00013/28.png)

当尝试访问一个已经被释放的引用计数对象是时，会导致 `IllegalReferenceCountException` 异常。

>__谁负责释放？__
通常来说，最后访问它的人需要释放它。

 
