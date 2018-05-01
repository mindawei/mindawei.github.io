---
title: 《Netty-in-Action》笔记（11）
date: 2018-02-19 14:31:52
tags: [笔记,Netty,Java] 
categories: Netty
---

# Provided ChannelHandlers and codecs
This chapter covers
* Securing Netty applications with SSL/TLS
* Building Netty HTTP/HTTPS applications
* Handling idle connections and timeouts
* Decoding delimited and length-based protocols
* Writing big data

<!-- more -->

## Securing Netty applications with SSL/TLS
To support SSL/TLS, Java provides the package `javax.net.ssl`, whose classes `SSLContext` and `SSLEngine` make it quite straightforward to implement decryption and encryption. Netty leverages this API by way of a `ChannelHandler` implementation named `SslHandler`, which employs an `SSLEngine` internally to do the actual work.

>__Netty’s OpenSSL/SSLEngine implementation__
>Netty also provides an `SSLEngine` implementation that uses the OpenSSL toolkit(www.openssl.org). This class, `OpenSslEngine`, offers better performance than the `SSLEngine` implementation supplied by the JDK.
>
>Netty applications (clients and servers) can be configured to use `OpenSslEngine` by default if the OpenSSL libraries are available. If not, Netty will fall back to the JDK implementation. For detailed instructions on configuring OpenSSL support, please see the Netty documentation at `http://netty.io/wiki/forked-tomcat-native.html#wikih2-1`.
>
>Note that the SSL API and data flow are identical whether you use the JDK’s `SSLEngine` or Netty’s `OpenSslEngine`.

Figure 11.1 shows data flow using `SslHandler`.
![Figure 11.1 Data flow through SslHandler for decryption and encryption](/images/00019/01.png "Figure 11.1 Data flow through SslHandler for decryption and encryption")

Listing 11.1 shows how an `SslHandler` is added to a `ChannelPipeline` using a `ChannelInitializer`.
![](/images/00019/02.png) 

The `SslHandler` has some useful methods, as shown in the following table. 
![](/images/00019/03.png) 

## Building Netty HTTP/HTTPS applications
### HTTP decoder, encoder, and codec

Figures 11.2 shows the methods for HTTP requests .
![Figure 11.2 HTTP request component parts](/images/00019/04.png "Figure 11.2 HTTP request component parts") 

Figures 11.3 shows the methods for HTTP responses.
![Figure 11.3 HTTP response component parts](/images/00019/05.png "Figure 11.3 HTTP response component parts") 

The following table gives an overview of the HTTP decoders and encoders that handle and produce these messages.
![](/images/00019/06.png) 

All types of HTTP messages (`FullHttpRequest`, `LastHttpContent`, and those shown in the above list) implement the `HttpObject` interface.

The class `HttpPipelineInitializer` in the next listing shows how simple it is to add HTTP support to your application.
![](/images/00019/07.png) 

### HTTP message aggregation

Netty provides an aggregator that merges message parts into `FullHttpRequest` and `FullHttpResponse` messages. This way you always see the full message contents. The following list shows how this is done.
![](/images/00019/08.png) 

### HTTP compression
Netty provides `ChannelHandler` implementations for compression and decompression that support both `gzip` and `deflate` encodings.

>__HTTP request header__
>The client can indicate supported encryption modes by supplying the followingheader:
```
 GET /encrypted-area HTTP/1.1
 Host: www.example.com
 Accept-Encoding: gzip, deflate
```
>Note, however, that the server isn’t obliged to compress the data it sends.

An example is shown in the following listing.
![](/images/00019/09.png) 

>__Compression and dependencies__
If you’re using JDK 6 or earlier, you’ll need to add JZlib (www.jcraft.com/jzlib/) to the CLASSPATH to support compression. For Maven, add the following dependency:
```
 <dependency>
 <groupId>com.jcraft</groupId>
 <artifactId>jzlib</artifactId>
 <version>1.1.3</version>
 </dependency>
```

### Using HTTPS
The following listing shows that enabling HTTPS is only a matter of adding an `SslHandler` to the mix.
![](/images/00019/10.png) 

### WebSocket
WebSockets provide a true _bidirectional_ exchange of data between client and server.

Figure 11.4 gives a general idea of the WebSocket protocol.  In this scenario the communication starts as plain HTTP and upgrades to bidirectional WebSocket.
![Figure 11.4 WebSocket protocol](/images/00019/11.png "Figure 11.4 WebSocket protocol")

As shown in the following table, `WebSocketFrames` can be classed as data or control frames.
![](/images/00019/12.png) 

>__Secure WebSocket__
To add security to WebSocket, simply insert the `SslHandler` as the first `ChannelHandler` in the pipeline.

## Idle connections and timeouts
Detecting idle connections and timeouts is essential to freeing resources in a timely manner. This is such a common task that Netty provides several `ChannelHandler` implementations just for this purpose. The following table shows these.
![](/images/00019/13.png) 

Let’s take a closer look at `IdleStateHandler`, the one most used in practice. The following list shows this.
![](/images/00019/14.png) 

## Decoding delimited and length-based protocols
### Delimited protocols
The decoders listed in the following table will help you to define custom decoders that can extract frames delimited by any sequence of tokens.
![](/images/00019/15.png)

Figure 11.5 shows how frames are handled when delimited by the end-of-line sequence `\r\n` (carriage return + line feed).
![Figure 11.5 Frames delimited by line endings](/images/00019/16.png "Figure 11.5 Frames delimited by line endings")

The following listing shows how you can use LineBasedFrameDecoder to handle the case shown in figure 11.5.
![](/images/00019/17.png)

### Length-based protocols

The following table lists the two decoders Netty provides for handling length-based protocols.
![](/images/00019/18.png)

Figure 11.6 shows the operation of a `FixedLengthFrameDecoder` that has been constructed with a frame length of 8 bytes.
![Figure 11.6 Decoding a frame length of 8 bytes](/images/00019/19.png "Figure 11.6 Decoding a frame length of 8 bytes")

`LengthFieldBasedFrameDecoder` determines the frame length from the header field and extracts the specified number of bytes from the data stream.

Figure 11.7 shows an example where the length field in the header is at offset 0 and has a length of 2 bytes.
![Figure 11.7 Message with variable frame size encoded in the header](/images/00019/20.png "Figure 11.7 Message with variable frame size encoded in the header")

The following list shows the use of a constructor whose three arguments are `maxFrameLength`, `lengthFieldOffset`, and `lengthFieldLength`.
![](/images/00019/21.png)

## Writing big data
This listing shows how you can transmit a file’s contents using zero-copy by creating a `DefaultFileRegion` from a `FileInputStream` and writing it to a `Channel`.
![](/images/00019/22.png)

This example applies only to the direct transmission of a file’s contents, excluding any processing of the data by the application. In cases where you need to copy the data from the file system into user memory, you can use `ChunkedWriteHandler`, which provides support for writing a large data stream asynchronously without incurring high memory consumption.

The key is interface `ChunkedInput<B>`, where the parameter `B` is the type returned by the method `readChunk()`. Four implementations of this interface are provided, as listed in the following table.
![](/images/00019/23.png)

The following list illustrates the use of `ChunkedStream`, the implementation most often used in practice. 
![](/images/00019/24.png)

## Serializing data
### JDK serialization
If your application has to interact with peers that use `ObjectOutputStream` and `ObjectInputStream`, and compatibility is your primary concern, then JDK serialization is the right choice. The following table lists the serialization classes that Netty provides for interoperating with the JDK.
![](/images/00019/25.png)

### Serialization with JBoss Marshalling
If you are free to make use of external dependencies, JBoss Marshalling is ideal: It’s up to three times faster than JDK Serialization and more compact. The overview on the JBoss Marshalling homepage defines it this way:
>JBoss Marshalling is an alternative serialization API that fixes many of the problems found in the JDK serialization API while remaining fully compatible with java.io.Serializable and its relatives, and adds several new tunable parameters and additional features, all of which are pluggable via factory configuration (externalizers, class/instance lookup tables, class resolution, and object replacement, to name a few).

Netty supports JBoss Marshalling with the two decoder/encoder pairs shown in the following table.  The first set is compatible with peers that use only JDK Serialization. The second, which provides maximum performance, is for use with peers that use JBoss Marshalling.
![](/images/00019/26.png)

The following listing shows how to use `MarshallingDecoder` and `MarshallingEncoder`.
![](/images/00019/27.png)
 
### Serialization via Protocol Buffers
The following table shows the `ChannelHandler` implementations Netty supplies for protobuf support.
![](/images/00019/28.png)

Here again, using protobuf is a matter of adding the right `ChannelHandler` to the `ChannelPipeline`, as shown in the following list.
![](/images/00019/29.png)


