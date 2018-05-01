---
title: 《Netty-in-Action》笔记（13）
date: 2018-02-22 16:00:48
tags: [笔记,Netty,Java] 
categories: Netty
---

# Broadcasting events with UDP
This chapter covers
* An overview of UDP
* A sample broadcasting application

<!-- more -->

## UDP basics
UDP is much faster than TCP: all overhead of handshaking and message management has been eliminated. Clearly, UDP is a good fit for applications that can handle or tolerate lost messages, unlike those that handle financial transactions

## UDP broadcast
All of our examples so far have utilized a transmission mode called _unicast_, defined as the sending of messages to a single network destination identified by a unique address.

UDP provides additional transmission modes for sending a message to multiple recipients:
* _Multicast_ — Transmission to a defined group of hosts
* _Broadcast_ — Transmission to all of the hosts on a network (or a subnet)

## The UDP sample application
>__PUBLISH/SUBSCRIBE__
Applications like syslog are typically classified as publish/subscribe: a producer or service publishes the events, and multiple clients subscribe to receive them.

Figure 13.1 presents a high-level view of the overall system, which consists of a broadcaster and one or more event monitors. 
![Figure 13.1 Broadcast  system overview](/images/00021/01.png "Figure 13.1 Broadcast  system overview") 

## The message POJO: LogEvent
Listing 13.1 shows the details of this the POJO `LogEvent`.
![](/images/00021/02.png) 

## Writing the broadcaster
The primary ones we’ll be using are the message containers and Channel types listed in the following table.
![](/images/00021/03.png) 

Figure 13.2 shows the broadcasting of three log entries, each one via a dedicated `DatagramPacket`.
![Figure 13.2 Log entries sent via DatagramPackets](/images/00021/04.png "Figure 13.2 Log entries sent via DatagramPackets") 

Figure 13.3 represents a high-level view of the `ChannelPipeline` of the `LogEventBroadcaster`, showing how `LogEvents` flow through it.
![](/images/00021/05.png) 

The next listing shows our customized version of `MessageToMessageEncoder`, which performs the conversion just described.
![](/images/00021/06.png) 

_netcat_ is perfect for basic testing of this application; it just listens on a specified port and prints all data received to standard output. Set it to listen for UDP data on port 9999 as follows:
```
$ nc -l -u 9999
```

## Writing the monitor
This program will 
1. Receive UDP `DatagramPackets` broadcast by the `LogEventBroadcaster`
2. Decode them to `LogEvent` messages
3. Write the `LogEvent` messages to `System.out`

Figure 13.4 depicts the `ChannelPipeline` of the `LogEventMonitor` and shows how `LogEvents` will flow through it.
![Figure 13.4 LogEventMonitor](/images/00021/07.png "Figure 13.4 LogEventMonitor")

The following listing shows `LogEventDecoder`.
![](/images/00021/08.png) 
 
The following listing shows `LogEventHandler`.
![](/images/00021/09.png) 

The following listing shows ` LogEventMonitor`.
![](/images/00021/10.png) 

