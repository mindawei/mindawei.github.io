---
title: HTTP变化介绍
date: 2018-03-13 13:30:46
tags: [网络,Http] 
categories: 网络
---

本文是对 HTTP 变化的介绍，是对以下资料的摘录：
* [《Hypertext Transfer Protocol》](https://en.wikipedia.org/wiki/Hypertext_Transfer_Protocol)
* [《HTTP,HTTP2.0,SPDY,HTTPS看这篇就够了》](https://www.nihaoshijie.com.cn/index.php/archives/630/)
* [《HTTP/2.0 相比1.0有哪些重大改进？》](https://www.zhihu.com/question/34074946)
* 本科课件

<!-- more -->

# HTTP 概览
HTTP 是 Hypertext Transfer Protocol 的简称，中文是超文本传输协议。它是一个应用层协议，可以支持分布式的、协作的、具有富文本信息的系统。（定义来自[维基百科](https://en.wikipedia.org/wiki/Hypertext_Transfer_Protocol)）

HTTP 的发展过程如下所示：
* 1989年，HTTP 起源于 Tim Berners-Lee 在 CERN 的工作。HTTP 的标准化发展过程是由因特网工程任务组（Internet Engineering Task Force，IETF）和万维网联盟（World Wide Web Consortium，W3C）共同推进的，主要是一系列的 RFC 标准（Requests for Comments，RFCs）。

* 1991年，HTTP 第一个标准文档 [HTTP V0.9](https://www.w3.org/Protocols/HTTP/AsImplemented.html) 发布。

* 1996年，Dave Raggett 领导了 HTTP 工作组（HTTP Working Group，HTTP WG）进一步完善了 HTTP 的功能，在 [RFC 1945](https://tools.ietf.org/html/rfc1945) 中发布了 HTTP V1.0。

* 1997年，一个使用更加广泛的版本 HTTP/1.1 在 [RFC 2068](https://tools.ietf.org/html/rfc2068) 中被提出。1999年发布的 [RFC 2616](https://tools.ietf.org/html/rfc2616) 对上一个版本进行了更新。2014年又发布了6个规范进行了一步的升级，具体包括：
>[RFC 7230](https://tools.ietf.org/html/rfc7230), HTTP/1.1: Message Syntax and Routing（消息语法和路由）
>[RFC 7231](https://tools.ietf.org/html/rfc7231), HTTP/1.1: Semantics and Content（语义和内容）
>[RFC 7232](https://tools.ietf.org/html/rfc7232), HTTP/1.1: Conditional Requests（条件请求）
>[RFC 7233](https://tools.ietf.org/html/rfc7233), HTTP/1.1: Range Requests（范围请求）
>[RFC 7234](https://tools.ietf.org/html/rfc7234), HTTP/1.1: Caching（缓存）
>[RFC 7235](https://tools.ietf.org/html/rfc7235), HTTP/1.1: Authentication（认证）

* 2015年，一个新的标准 [HTTP/2](http://httpwg.org/specs/rfc7540.html) 在 [RFC 7540](https://tools.ietf.org/html/rfc7540) 中发布，[它建立在 TLS 层上，使用 ALPN 进行扩展](https://tools.ietf.org/html/rfc7301)，现在已经被很多 web 服务器和浏览器支持了。

![HTTP版本变化](/images/00025/01.png "HTTP版本变化")

# HTTP 基本优化
影响 HTTP 响应速度的因素主要有：__带宽__和__延迟__。由于__带宽__是外在环境，并且这些年带宽速度得到了极大提升，所以不再详细介绍。 而影响__延迟__的因素又包括：
* __浏览器阻塞__（HOL blocking）：浏览器会因为一些原因阻塞请求。浏览器客户端在同一时间，针对同一域名下的请求有一定数量限制。超过限制数目的请求会被阻塞。具体如下表所示（[表格来源](http://www.stevesouders.com/blog/2008/03/20/roundup-on-parallel-connections/)）：

浏览器| HTTP/1.1 | HTTP/1.0
--|--|--
IE 6,7 |2|4
IE 8 |6|6 
Firefox 2 |2|8
Firefox 3 |6|6
Safari 3,4 |4|4
Chrome 1,2 |6|？
Chrome 3 |4|4
Chrome 4+ |6|？
iPhone 2 |4|？
iPhone 3 |6|？
iPhone 4 |4|？
Opera 9.63,10.00alpha |4|4
Opera 10.51+ |8|？



* __DNS 查询__（DNS Lookup）：浏览器需要知道目标服务器的 IP 才能建立连接。将域名解析为 IP 的这个系统就是 DNS。这个通常可以利用DNS缓存结果来达到减少这个时间的目的。
* __建立连接__（Initial connection）：HTTP 是基于 TCP 协议的，浏览器最快也要在第三次握手时才能捎带 HTTP 请求报文，达到真正的建立连接，但是这些连接无法复用会导致每次请求都经历三次握手和[慢启动](https://en.wikipedia.org/wiki/TCP_congestion_control#Slow_start)。三次握手在高延迟的场景下影响较明显，[慢启动](https://en.wikipedia.org/wiki/TCP_congestion_control#Slow_start)则对文件类大请求影响较大。
![TCP的三次握手四次挥手](/images/00025/02.png "TCP的三次握手四次挥手")

# HTTP 1.1
[HTTP 1.1](https://www.w3.org/Protocols/rfc2616/rfc2616.html) 在1999年才开始广泛应用于现在的各大浏览器网络请求中，同时 HTTP 1.1 也是当前使用最为广泛的 HTTP 协议。HTTP 1.0 的新特性有：
* __长连接__。HTTP 1.1 支持长连接（PersistentConnection）和 请求的流水线（Pipelining）处理，在一个TCP连接上可以传送多个HTTP请求和响应，减少了建立和关闭连接的消耗和延迟，在 HTTP 1.1 中默认开启 Connection： keep-alive，一定程度上弥补了 HTTP 1.0 每次请求都要创建连接的缺点。
* __缓存处理__。在 HTTP 1.0 中主要使用 header 里的 If-Modified-Since、Expires 来做为缓存判断的标准，HTTP 1.1 则引入了更多的缓存控制策略，如：Entity tag、If-Unmodified-Since、If-Match、If-None-Match等，可以基于此提供更多的缓存控制策略。
* __带宽优化及网络连接的使用__。HTTP 1.0 中，存在一些浪费带宽的现象，如：客户端只是需要某个对象的一部分，而服务器却将整个对象送过来了，并且不支持断点续传功能，HTTP 1.1 则在请求头引入了 range 头域，它允许只请求资源的某个部分，即返回码是206（Partial Content），这样就方便了开发者自由选择资源区域，便于充分利用带宽和连接。
* __错误通知的管理__。在 HTTP 1.1 中新增了24个错误状态响应码，如：409（Conflict）表示请求的资源与资源的当前状态发生冲突；410（Gone）表示服务器上的某个资源被永久性的删除。
* __Host头处理__。在 HTTP 1.0 中认为每台服务器都绑定一个唯一的IP地址，因此请求消息中的URL并没有传递主机名（hostname）。但随着虚拟主机技术的发展，在一台物理服务器上可以存在多个虚拟主机（Multi-homed Web Servers），并且它们共享一个IP地址。HTTP 1.1 的请求消息和响应消息都应支持Host头域，且请求消息中如果没有 Host 头域会报告一个错误（400 Bad Request）。
* __缓冲区指针__。实现缓冲区指针允许客户和服务器的缺省缓冲区算法可以被调用或优化。* __主机标题__。HTTP 1.1 协议允许多重主机名与一个单独的IP地址相关联。这样就不用给一个驻留许多虚拟服务器的 Web 服务器配置多个IP地址了。这个主机标题可以用来确定请求应该被导向哪个虚拟服务器。
* __PUT和DELETE选项__。这些命令允许一个远程管理者通过使用一个标准的 Web 浏览器来记入或删除一些内容。
* __HTTP重定向__。当原始的主页不能访问或被删除时，这个特性允许一个管理者将一个用户重定向到一个备选的主页或 Web 站点。

# HTTP 1.0 和 1.1 的一些问题
* HTTP 1.x 在传输数据时，每次都需要重新建立连接，这增加了大量的延迟时间，在移动端更为突出。
* HTTP 1.x 在传输数据时，所有传输的内容都是明文，客户端和服务器端都无法验证对方的身份，这在一定程度上无法保证数据的安全性。
* HTTP 1.x 在使用时，header里携带的内容过大，在一定程度上增加了传输的成本，并且每次请求header基本不怎么变化。在移动端用户流量较大时，问题更为突出。
* HTTP 1.x 虽然通过支持keep-alive来弥补多次创建连接产生的延迟，但是keep-alive使用多了同样会给服务端带来大量的性能压力，并且对于单个文件被不断请求的服务(如图片存放网站)，keep-alive可能会极大的影响性能，因为它在文件被请求之后，还长时间地保持了不必要的连接。


# HTTPS
为了解决传输的安全问题，Netscape 在1994年创建了 HTTPS ，并应用在浏览器中。最初，HTTPS 是与 SSL 一起使用的，后来 SSL 逐渐演变到 TLS （本质上两个是一个东西）。最新的 HTTPS 在2000年5月公布的RFC 2818正式确定下来。

简单来说，HTTPS 就是安全版的 HTTP，并且由于当今时代对安全性要求更高，Chrome 和 Firefox 都大力支持网站使用 HTTPS，苹果也在 IOS 10系统中强制 App 使用 HTTPS 来传输数据，由此可见 HTTPS 势在必行。

__HTTPS 与 HTTP 的一些区别__
* HTTPS 协议需要到 CA 申请证书。
* HTTP 是超文本传输协议，信息是明文传输，HTTPS 则是具有安全性的 SSL 加密传输协议。可进行身份认证，比 HTTP 协议安全。
* HTTP 和 HTTPS 使用的是完全不同的连接方式，用的端口也不一样，HTTP 是80，HTTPS 是443。

HTTPS 由于增加了 SSL 的握手过程，会有一定的性能损失。此外，由于有较多的秘钥算法计算，需要关注服务端的 CPU 压力。推荐一个PPT[《淘宝HTTPS探索》](http://velocity.oreilly.com.cn/2015/ppts/lizhenyu.pdf)。

# SPDY
2012年 Google 提出了 [SPDY](https://tools.ietf.org/html/draft-mbelshe-httpbis-spdy-00) 的方案，大家才开始从正面看待和解决老版本 HTTP 协议本身的问题，SPDY 综合了 HTTPS 和 HTTP 两者的优点于一体，主要特点如下所示：
* __降低延迟__。针对 HTTP 高延迟的问题，SPDY优雅的采取了多路复用（multiplexing）。多路复用通过多个请求 stream 共享一个 TCP 连接的方式，解决了 HOL blocking 的问题，降低了延迟同时提高了带宽的利用率。
* __请求优先级（request prioritization）__。多路复用带来一个新的问题是，在连接共享的基础之上有可能会导致关键请求被阻塞。SPDY 允许给每个 request 设置优先级，这样重要的请求就会优先得到响应。如:浏览器加载首页，首页的html内容应该优先展示，之后才是各种静态资源文件，脚本文件等加载，这样可以保证用户能第一时间看到网页内容。
* __header压缩__。前面提到 HTTP1.x 的 header 很多时候都是重复多余的。选择合适的压缩算法可以减小包的大小和数量。
* __基于 HTTPS 的加密协议传输__。大大提高了传输数据的可靠性。
* __服务端推送（server push）__。采用了 SPDY 的网页，例如：网页有一个 sytle.css 的请求，在客户端收到 sytle.css 数据的同时，服务端会将 sytle.js 的文件推送给客户端，当客户端再次尝试获取 sytle.js 时就可以直接从缓存中获取到，不用再发请求了。

SPDY 构成图如下所示：
![SPDY构成](/images/00025/03.png "SPDY构成")

# HTTP 2.0
[HTTP 2.0](http://httpwg.org/specs/rfc7540.html) 可以说是 SPDY 的升级版（其实原本也是基于 SPDY 设计的）。但 HTTP2.0 跟 SPDY 仍有不同的地方，主要是：
* HTTP 2.0 支持明文 HTTP 传输，而 SPDY 强制使用 HTTPS。
* HTTP 2.0 消息头的压缩算法采用 [HPACK](https://http2.github.io/http2-spec/compression.html)，而 SPDY 采用的是 [DEFLATE](https://zh.wikipedia.org/wiki/DEFLATE)。

HTTP 2.0 与 HTTP 1.0 的速度对比如下（[演示网站](https://http2.akamai.com/demo)）：
![速度对比](/images/00025/04.png "速度对比")

HTTP 2.0 的一些新的特性包括：
* __多路复用（MultiPlexing），即连接共享__。 每一个 request 都是是用作连接共享机制的。一个 request 对应一个id，这样一个连接上可以有多个request，每个连接的 request 可以随机的混杂在一起，接收方可以根据 request 的 id 将 request 归属到各自不同的服务端请求里面。多路复用原理图如下所示：
![多路复用](/images/00025/05.png "多路复用")

![Http1.1和Http2的对比](/images/00025/06.jpg "Http1.1和Http2的对比")

* __新的二进制格式（Binary Format）__，HTTP1.x 的解析是基于文本。基于文本协议的格式解析存在天然缺陷，文本的表现形式有多样性，要做到健壮性考虑的场景必然很多，二进制则不同，只认0和1的组合。基于这种考虑 HTTP 2.0 的协议解析决定采用二进制格式，实现方便且健壮。实现关键之一就是在应用层（HTTP/2）和传输层（TCP 或 UDP）之间增加一个二进制分帧层。
![二进制分帧](/images/00025/07.jpg "二进制分帧")

* __Header 压缩__。前面提到过 HTTP1.x 的 header 带有大量信息，而且每次都要重复发送，HTTP2.0 使用 encoder 来减少需要传输的 header 大小，通讯双方各自 cache 一份 header fields 表，既避免了重复 header 的传输，又减小了需要传输的大小。
* __服务端推送（server push）__。同SPDY一样，HTTP 2.0 也具有 server push 功能。目前，有大多数网站已经启用 HTTP 2.0 了。
 
更多关于 HTTP 2.0 资料如下：
* [《HTTP2 奇妙日常》](http://www.alloyteam.com/2015/03/http2-0-di-qi-miao-ri-chang/)。
* HTTP 2.0 的[官方网站](https://http2.akamai.com/)。
* [《HTTP2 讲解》](https://www.gitbook.com/book/ye11ow/http2-explained/details)。
* [《HTTP/2 for Front-End Developers》](https://www.mnot.net/talks/h2fe/)。
* [《7 Tips for Faster HTTP/2 Performance》](https://www.nginx.com/blog/7-tips-for-faster-http2-performance/)。