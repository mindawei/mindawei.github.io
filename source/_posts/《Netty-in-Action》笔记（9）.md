---
title: 《Netty-in-Action》笔记（9）
date: 2018-02-18 19:00:27
tags: [笔记,Netty,Java] 
categories: Netty
---

# Unit testing
This chapter covers
* Unit testing
* Overview of `EmbeddedChannel`
* Testing `ChannelHandlers` with `EmbeddedChannel`

<!-- more -->

## Overview of EmbeddedChannel
Netty provides what it calls an _embedded transport_ for testing `ChannelHandlers`. This transport is a feature of a special `Channel` implementation, `EmbeddedChannel`, which provides a simple way to pass events through the pipeline.

The relevant methods of EmbeddedChannel are listed in the following table.
![](/images/00017/01.png)

Figure 9.1 shows how data flows through the `ChannelPipeline` using the methods of `EmbeddedChannel`. 
![Figure 9.1 EmbeddedChannel data flow](/images/00017/02.png "Figure 9.1 EmbeddedChannel data flow")

## Testing ChannelHandlers with EmbeddedChannel
### Testing inbound messages
Figure 9.2 represents a simple `ByteToMessageDecoder` implementation. 
![Figure 9.2 Decoding via FixedLengthFrameDecoder](/images/00017/03.png "Figure 9.2 Decoding via FixedLengthFrameDecoder")

The implementation of the decoder is shown in the following listing.
![](/images/00017/04.png)

This listing shows a test of the preceding code using `EmbeddedChannel`.
![](/images/00017/11.png)

### Testing outbound messages
Figure 9.3 shows the logic.
![](/images/00017/05.png)

The next listing implements this logic, illustrated in figure 9.3. 
![](/images/00017/06.png)

The next listing tests the code using `EmbeddedChannel`.
![](/images/00017/07.png)

## Testing exception handling
In figure 9.4 the maximum frame size has been set to 3 bytes. If the size of a frame exceeds that limit, its bytes are discarded and a `TooLongFrameException` is thrown. 
![Figure 9.4 Decoding via FrameChunkDecoder](/images/00017/08.png "Figure 9.4 Decoding via FrameChunkDecoder")

The implementation is shown in the following listing.
![](/images/00017/09.png)

Again, we’ll test the code using `EmbeddedChannel`.
![](/images/00017/10.png)

The `try/catch` block used here is a special feature of `EmbeddedChannel`. If one of the `write*` methods produces a checked `Exception`, it will be thrown wrapped in a `RuntimeException`.


