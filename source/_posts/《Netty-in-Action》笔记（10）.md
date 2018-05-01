---
title: 《Netty-in-Action》笔记（10）
date: 2018-02-18 20:46:25
tags: [笔记,Netty,Java] 
categories: Netty
---

# The codec framework
This chapter covers
* An overview of decoders, encoders and codecs
* Netty’s codec classes

<!-- more -->

## What is a codec?
This conversion logic is handled by a _codec_, which consists of an encoder and a decoder, each of which transforms a stream of bytes from one format to another.

## Decoders
These classes cover two distinct use cases:
* Decoding bytes to messages — `ByteToMessageDecoder` and `ReplayingDecoder`
* Decoding one message type to another — `MessageToMessageDecoder`

### Abstract class ByteToMessageDecoder
The following table shows two most important methods in `ByteToMessageDecoder` API.
![](/images/00018/01.png)

To decode the byte stream, you’ll extend `ByteToMessageDecoder`. The design is illustrated in figure 10.1.
![Figure 10.1 ToIntegerDecoder](/images/00018/02.png "Figure 10.1 ToIntegerDecoder")

This listing shows the code for `ToIntegerDecoder`.
![](/images/00018/03.png)

>__Reference counting in codecs__
once a message has been encoded or decoded, it will automatically be released by a call to `ReferenceCountUtil.release(message)`. If you need to keep a reference for later use you can call `ReferenceCountUtil.retain(message)`. This increments the reference count, preventing the message from being released.

### Abstract class ReplayingDecoder
The full declaration of this class is
```
public abstract class ReplayingDecoder<S> extends ByteToMessageDecoder
```

The parameter `S` specifies the type to be used for state management, where `Void` indicates that none is to be performed. The following listing shows a reimplementation of `ToIntegerDecoder` based on `ReplayingDecoder`.
![](/images/00018/04.png)

Please take note of these aspects of `ReplayingDecoderBuffer`:
* Not all `ByteBuf` operations are supported. If an unsupported method is called, an `UnsupportedOperationException` will be thrown.
* `ReplayingDecoder` is slightly slower than `ByteToMessageDecoder`

Here’s a simple guideline: use `ByteToMessageDecoder` if it doesn’t introduce excessive complexity; otherwise, use `ReplayingDecoder`.

>__More decoders__
The following classes handle more complex use cases:
* `io.netty.handler.codec.LineBasedFrameDecoder` — This class, used internally by Netty, uses end-of-line control characters (`\n` or `\r\n`) to parse the message data.
* `io.netty.handler.codec.http.HttpObjectDecoder` — A decoder for HTTP data.

### Abstract class MessageToMessageDecoder
In this section we’ll explain how to convert between message formats using the abstract base class
```
public abstract class MessageToMessageDecoder<I> 
 extends ChannelInboundHandlerAdapter
```

The parameter `I` specifies the type of the input msg argument to `decode()`, which is the only method you have to implement. The following table shows the details of this method.
![](/images/00018/05.png)

The design of `IntegerToStringDecoder` is illustrated in figure 10.2.
![Figure 10.2 IntegerToStringDecoder](/images/00018/06.png "Figure 10.2 IntegerToStringDecoder")

The following listing is the implementation of `IntegerToStringDecoder`.
![](/images/00018/07.png)

>__HttpObjectAggregator__
For a more complex example, please examine the class `io.netty.handler.codec.http.HttpObjectAggregator`, which extends `MessageToMessageDecoder<HttpObject>`.

### Class TooLongFrameException
Netty provides a `TooLongFrameException`, which is intended to be thrown by decoders if a frame exceeds a specified size limit.
 
Listing 10.4 shows how a `ByteToMessageDecoder` can make use of `TooLongFrameException` to notify other `ChannelHandlers` in the `ChannelPipeline` about the occurrence of a frame-size overrun. 
![](/images/00018/08.png)


## Encoders
### Abstract class MessageToByteEncoder
The following table shows the `MessageToByteEncoder` API.
![](/images/00018/09.png)

You may have noticed that this class has only one method, while decoders have two. The reason is that decoders often need to produce a last message after the `Channel` has closed (hence the `decodeLast()` method).

Figure 10.3 shows a `ShortToByteEncoder`.
![Figure 10.3 ShortToByteEncoder](/images/00018/10.png "Figure 10.3 ShortToByteEncoder")

The implementation of `ShortToByteEncoder` is shown in the following listing.
![](/images/00018/11.png)

Netty provides several specializations of `MessageToByteEncoder` upon which you can base your own implementations. The class `WebSocket08FrameEncoder` provides a good practical example. You’ll find it in the package `io.netty.handler.codec.http.websocketx`.

### Abstract class MessageToMessageEncoder
The following table shows the `MessageToMessageEncoder` API.
![](/images/00018/12.png)

Figure 10.4 shows a `IntegerToStringEncoder`.
![Figure 10.4 IntegerToStringEncoder](/images/00018/13.png "Figure 10.4 IntegerToStringEncoder")

As shown in the next listing, the encoder adds a `String` representation of each outbound `Integer` to the `List`.
![](/images/00018/14.png)

For an interesting specialized use of `MessageToMessageEncoder`, look at the class `io.netty.handler.codec.protobuf.ProtobufEncoder`, which handles data formats defined by Google’s Protocol Buffers specification.

## Abstract codec classes
### Abstract class ByteToMessageCodec
The following table shows the `ByteToMessageCodec` API.
![](/images/00018/15.png)

Any request/response protocol could be a good candidate for using the `ByteToMessageCodec`. 

### Abstract class MessageToMessageCodec
`MessageToMessageCodec` is a parameterized class, defined as follows:
```
public abstract class MessageToMessageCodec<INBOUND_IN,OUTBOUND_IN>
```

The important methods are listed in the following table.
![](/images/00018/16.png)

>__WebSocket protocol__
`WebSocket` is a recent protocol that enables full bidirectional communications between web browsers and servers. 

### Class CombinedChannelDuplexHandler
`CombinedChannelDuplexHandler` declared as
```
public class CombinedChannelDuplexHandler
 <I extends ChannelInboundHandler, O extends ChannelOutboundHandler>
```

First, examine `ByteToCharDecoder` in the following listing.
![](/images/00018/17.png)
  
The following list shows `CharToByteEncoder`, which converts `Characters` back to bytes. 
![](/images/00018/18.png)

Now that we have a decoder and encoder, we’ll combine them to build up a codec. This listing shows how this is done.
![](/images/00018/19.png)

