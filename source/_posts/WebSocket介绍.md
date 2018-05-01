---
title: WebSocket介绍
date: 2018-03-14 10:09:44
tags: [网络,WebSocket] 
categories: 网络
---

本文是对 WebSocket 变化的介绍，是对以下资料的摘录：
* [维基：WebSocket](https://en.wikipedia.org/wiki/WebSocket)
* [《WebSocket 教程》](http://www.ruanyifeng.com/blog/2017/05/websocket.html)
* [RFC 6455：The WebSocket Protocol](https://tools.ietf.org/html/rfc6455)

<!-- more -->

# WebSocket 概览
WebSocket 是一个计算机间的通信协议，它能够在单个 TCP 链接上构建一个双工的交流通道。WebSocket 协议的标准是由 IETE 在 2011年的 [__RFC 6455__](https://tools.ietf.org/html/rfc6455) 中制定的。

WebSocket 是一个与 HTTP 并不相同的 TCP 的协议。WebSocket 和 HTTP 协议都是在7层网络模型（[__OSI model__](https://en.wikipedia.org/wiki/OSI_model)）中的，并且都依赖第4层的 TCP 协议。尽管两者并不相同，但 [__RFC 6455__](https://tools.ietf.org/html/rfc6455) 中声明 “WebSocket 是可以工作在 HTTP协议 的80和443端口之上的，并且能够支持 HTTP 代理和中介”，这使得 WebSocket 可以兼容 HTTP 协议。为了实现这个兼容目标，WebSocket 的[__握手（handshake）__](https://en.wikipedia.org/wiki/Handshaking)使用了 [__HTTP 的 Upgrade 头部__](https://en.wikipedia.org/wiki/HTTP/1.1_Upgrade_header)，从而实现从 HTTP 协议转换为 WebSocket 协议的目标。

WebSocket 能够建立客户端和服务器间的双向通信，目前很多浏览器都已经支持该协议了。同样，服务端也需要提供相应的支持。

WebSocket 协议标志是 `ws`（WebSocket）和 `wss`（WebSocket Secure）。
![协议标志](/images/00026/01.jpg "协议标志")  

# 为什么需要 WebSocket
由于 HTTP 是客户端发起的单向请求，对于聊天室这样需要服务端推送信息的场景就不是很适合。当然，客户端可以通过[__“轮询”__](https://www.pubnub.com/blog/2014-12-01-http-long-polling/)的方式来了解服务端的信息，但是这样的效率比较低，比较浪费资源（因为必须不停连接，或者 HTTP 连接始终打开）。

和 HTTP 不同，WebSocket 是一个全双工协议，属于[__服务器推送技术__](https://en.wikipedia.org/wiki/Push_technology)的一种。在 WebSocket 之前，在80端口可以通过 Comet 通道实现全双工。但是由于 TCP 握手和 HTTP 头部的开销，对于数据量不大的信息来说，这样的机制不是很高效。WebSocket 协议的目标就是解决这些问题并且提供相应的安全保障。

# 握手协议
WebSocket 建立连接的过程如下：
1. 客户端发送 WebSocket 握手请求（和 HTTP 一样, 每行要以 `\r\n` 结尾，最后要有一个额外的空行）。
```
GET /chat HTTP/1.1
Host: server.example.com
Upgrade: websocket
Connection: Upgrade
Sec-WebSocket-Key: x3JJHMbDL1EzLkh9GBhXDw==
Sec-WebSocket-Protocol: chat, superchat
Sec-WebSocket-Version: 13
Origin: http://example.com
```

2. 服务端回应握手请求。
```
HTTP/1.1 101 Switching Protocols
Upgrade: websocket
Connection: Upgrade
Sec-WebSocket-Accept: HSmrc0sMlYUkAGmm5OPpG2HaGWk=
Sec-WebSocket-Protocol: chat
```

可以看到，客户端发送的请求头部除了 `Upgrade` 外，还有 `Sec-WebSocket-Key` 字段，它是包含base64编码的随机字节，服务端需要在 `WebSocket-Accept` 字段中返回这个键的hash值。这是为了防止缓存代理重发之前的 WebSocket 会话。

通过类 HTTP 形式的握手形式，可以使得服务端在同一个端口处理 HTTP 或者 WebSocket 协议。一旦 WebSocket 握手完成，通信马上变换成一个与 HTTP 协议不同的双向通道。
![WebSocket协议过程](/images/00026/02.png "WebSocket协议过程") 

此外，从安全的角度来说，提供 `Origin` 标签是很有必要的，可以避免跨站点的 WebSocket 劫持攻击。

# 数据格式
全双工的通道建立之后，传输的数据将会以尽可能小的格式进行封装：一个小的头部，紧跟着的是有效数据载体 [__payload__](https://en.wikipedia.org/wiki/Payload_(computing))。WebSocket 传输的内容被称为“消息”，这个消息可以被划分成多个数据帧。这样在只有一部分初始数据（完整数据还未准备好）的时候就可以提前发送数据了。通过扩展，可以同时多路复用多个数据流（这样可以避免大负载数据对 socket 的独占使用）。
![数据格式](/images/00026/03.png "数据格式") 

__FIN__（1 bit）
表明消息是否是最后一帧。第一条消息也可能是最后一帧。

__RSV1, RSV2, RSV3__（每个都是1位）
必须是0，除非扩展定义了非0值的意义。如果收到非0值，并且没有具体的定义，那么接收端必须使连接失败。

__Opcode__（4位）
声明 “Payload data” 的含义。如果收到一个未知编码，那么接收端必须使连接失败。目前有以下这些值：
* `%x0` denotes a continuation frame，表明是一个持续帧。
* `%x1` denotes a text frame，表明是一个文本帧。
* `%x2` denotes a binary frame，表明是一个二进制帧。
* `%x3-7` are reserved for further non-control frames，为未来的非控制帧保留。
* `%x8` denotes a connection close，定义一个连接结束。
* `%x9` denotes a ping，表示 ping 操作。
* `%xA` denotes a pong，表示 pong 操作。
* `%xB-F` are reserved for further control frames，为未来的控制帧保留。

__ Mask__（1 bit）
定义 “Payload data” 是否是隐秘的。如果设置为1，那么 `masking-key` 中会提供一个值，并且可以根据 [5.3节](https://tools.ietf.org/html/rfc6455#section-5.3) 取消对应的标志。所有客户端发送到服务端的帧都需要把这位设置为1。

__Payload length__（7位 或 7+16位 或 7+64位）
表示 “Payload data” 的长度。分为这几种情况：
* 如果是 0-125，那么就是实际长度。
* 如果是 126，那么接下来的2个字节作为一个16位的无符号整数，表示实际的长度。
* 如果是 127，那么接下来的8个字节作为一个64位的无符号整数，表示实际的长度。

其中多字节表示的长度在网络中必须按序排列。值得注意的是，在所有例子中，长度必须按最短的表示方式表示。Payload 数据的长度是由 “Extension data” （扩展数据）和 “Extension data”（应用数据）组成的，当扩展数据的长度为0时，payload 数据的长度就是应用数据的长度。

__Masking-key__（0位或者4位）
所有从客户端发送到服务端的数据帧都需要有一个32位的 `Masking-key`。当 `mask` 位被设置为1的时候，这个域存在；当 `mask` 位被设置为0的时候，这个域不存在。可以看 [5.3节](https://tools.ietf.org/html/rfc6455#section-5.3) 来进一步了解。

__Payload data__（x+y位）
由 "Extension data"（扩展数据）和 "Application data"（应用数据）组成。
* __Extension data__（x位）：扩展数据默认是0位，除非实现了一个扩展协议。每一个扩展的协议必须说明扩展数据的长度、长度是如何计算的、以及在握手阶段扩展是如何约定的。如果该部分数据存在，那么将会被记录到总的 payload 长度中。
* __Application data__（y位）：应用数据在扩展数据之后，应用数据的长度等于 payload 的长度减去扩展数据的长度。

![WebSocket 数据格式](/images/00026/04.png "WebSocket 数据格式") 

# 其它资料
进一步了解可以参考这些文档：
* [w3cschool](https://www.w3cschool.cn/html5/html5-websocket.html)
* [HTML5 WebSocket](http://www.runoob.com/html/html5-websocket.html)
* [WebSocket - Web APIs | MDN](https://developer.mozilla.org/en-US/docs/Web/API/WebSocket)
* [websocket.org](http://websocket.org/#)
* [WebSocket 的实现比较](https://en.wikipedia.org/wiki/Comparison_of_WebSocket_implementations)
* [WebSocket 是什么原理？为什么可以实现持久连接？](https://www.zhihu.com/question/20215561)

推荐一款非常特别的 WebSocket 服务器：[Websocketd](http://websocketd.com/)。它的最大特点，就是后台脚本不限语言，标准输入（stdin）就是 WebSocket 的输入，标准输出（stdout）就是 WebSocket 的输出。


 

