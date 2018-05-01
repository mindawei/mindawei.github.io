---
title: 《Netty-in-Action》笔记（12）
date: 2018-02-22 11:14:49
tags: [笔记,Netty,Java] 
categories: Netty
---

# WebSocket
This chapter covers
* The concept of a real-time web
* The WebSocket protocol
* Building a WebSocket-based chat room server with Netty

<!-- more -->

So for now we’ll accept the following non-authoritative description from Wikipedia as adequate: 
>The _real-time_ web is a network web using technologies and practices that enable users to receive information as soon as it is published by its authors, rather than requiring that they or their software check a source periodically for updates. 

## Introducing WebSocket
The WebSocket protocol was designed from the ground up to provide a practical solution to the problem of bidirectional data transmission on the web, allowing client and server to transmit messages at any time and, consequently, requiring them to handle message receipt asynchronously. 

## Our example WebSocket application
Figure 12.1 illustrates the application logic:
1 A client sends a message.
2 The message is broadcast to all other connected clients.
![Figure 12.1 WebSocket application logic](/images/00020/01.png "Figure 12.1 WebSocket application logic") 

## Adding WebSocket support
A mechanism known as the _upgrade handshake_ is used to switch from standard HTTP or HTTPS protocol to WebSocket.

Figure 12.2 illustrates the server logic, which, as always in Netty, will be implemented by a set of `ChannelHandlers`. 
![Figure 12.2 Server logic](/images/00020/02.png "Figure 12.2 Server logic")
 
### Handling HTTP requests
The following list shows the code for `HttpRequestHandler`, which extends `SimpleChannelInboundHandler` for `FullHttpRequest` messages. Notice how the implementation of `channelRead0()` forwards any requests for the URI `/ws`.
 
This represents the first part of the chat server, which manages pure HTTP requests and responses. Next we’ll handle the WebSocket frames, which transmit the actual chat messages. 

>__WEBSOCKET FRAMES__
>WebSockets transmit data in frames, each of which represents a part of a message. A complete message may consist of many frames. 
 
### Handling WebSocket frames
The WebSocket RFC, published by the IETF, defines six frames; Netty provides a POJO implementation for each of them. The following table lists the frame types and describes their use.
![](/images/00020/03.png) 

Our chat application will use the following frame types:
* `CloseWebSocketFrame`
* `PingWebSocketFrame`
* `PongWebSocketFrame`
* `TextWebSocketFrame`

`TextWebSocketFrame` is the only one we actually need to handle. In conformity with the WebSocket RFC, Netty provides a `WebSocketServerProtocolHandler` to manage the others.

The following listing shows our `ChannelInboundHandler` for `TextWebSocketFrames`, which will also track all the active WebSocket connections in its ChannelGroup.
![](/images/00020/04.png)

### Initializing the ChannelPipeline
The following listing shows the code for the resulting `ChatServerInitializer`.
![](/images/00020/05.png)

The call to `initChannel()` sets up the `ChannelPipeline` of the newly registered `Channel` by installing all the required `ChannelHandlers`. These are summarized in the following table, along with their individual responsibilities.
![](/images/00020/06.png)

The state of the pipeline before the upgrade is illustrated in figure 12.3. 
![Figure 12.3 ChannelPipeline before WebSocket upgrade](/images/00020/07.png "Figure 12.3 ChannelPipeline before WebSocket upgrade")

Figure 12.4 shows the `ChannelPipeline` after these operations have completed.
![Figure 12.4 ChannelPipeline after WebSocket upgrade](/images/00020/08.png "Figure 12.4 ChannelPipeline after WebSocket upgrade")

### Bootstrapping
The final piece of the picture is the code that bootstraps the server and installs the `ChatServerInitializer`. This will be handled by the `ChatServer` class.

## Testing the application
We’ll use the following Maven command to build and start the server:
```
mvn -PChatServer clean package exec:exec
```

To use a different port, you can either edit the value in the file or override it with a System property:
```
mvn -PChatServer -Dport=1111 clean package exec:exec
```

Figure 12.5 shows the UI in the Chrome browser.
![Figure 12.5 WebSocket ChatServer demonstration](/images/00020/09.png "Figure 12.5 WebSocket ChatServer demonstration")

The figure shows two connected clients. The first is connected using the interface at the top. The second client is connected via the Chrome browser’s command line at the bottom. You’ll notice that there are messages sent from both clients, and each message is displayed to both.

### What about encryption?
The following listing shows how this can be done by extending our `ChatServerInitializer` to create a `SecureChatServerInitializer`.
![](/images/00020/10.png)

The final step is to adapt the `ChatServer` to use the `SecureChatServerInitializer` so as to install the `SslHandler` in the pipeline. This gives us the `SecureChatServer` shown here.
![](/images/00020/11.png)




