---
title: NSQ学习 逻辑梳理
date: 2019-12-17 16:27:00
mathjax: true
tags:
- NSQ
---

# 消费者连接NSQD或NSQLookupd
消费者可以连接多个`NSQLookupd`，消费者从每一个`NSQLookupd`查询拥有需要消费的`Topic`的`NSQD`，并与查询得到的每一个`NSQD`建立连接。
每当`NSQD`中`Topic`或`Channle`出现了变动，`NSQD`会通知每一个与它相连的`NSQLookupd`。

# 一条消息的生命周期
一条消息首先被生产者投递到`NSQD`的`Topic`，然后由`Topic.messagePump`这个方法分发给`Topic`的`Channel`。

`nsq/nsqd/protocol_v2.go`中的`protocolV2.IOLoop`方法一边根据客户端的接收情况把`Channel`中的消息发给客户端（`protocolV2.messagePump`），一遍处理来自客户端的指令如`REQ`、`FIN`等。
消息发给客户端之前，首先push到`InFlightQueue`这个优先队列中，这个队列可以表示正在处理中的消息。
`InFlightQueue`是最小堆，用于比较的属性是过期时刻。
每当消费者发送表示执行成功的`FIN`指令给`NSQD`时，`NSQD`就会把执行成功的消息从对应`Channel`的`InFlightQueue`删除；
如果直到超时还未收到消费者的`FIN`指令，那么对应的消息会从`InFlightQueue`删除，然后重新投递给`Channel`；
如果收到`REQ`指令，说明对应消息需要重新执行，这条消息会从`InFlightQueue`删除然后进入`DeferredQueue`，等到需要被执行的时刻再重新投递给`Channel`。
`DeferredQueue`也是最小堆，用于比较的属性是消息被执行的时刻。

![](https://img2018.cnblogs.com/blog/1224734/201912/1224734-20191217162714868-2037097168.png)

# 启动和关闭
## DeferredQueue和InFlightQueue
`NSQD`启动时，通过`Metadata`生成`Topic`和`Channel`，并根据`Topic`和`Channel`找到对应的文件队列。

`NSQD`关闭时，每一个`Channel`的`memoryMsgChan`的消息会被写到对应的文件队列里，`inFlightMessages`和`deferredMessages`同上。