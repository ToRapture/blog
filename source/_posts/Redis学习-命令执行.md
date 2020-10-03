---
title: Redis学习 命令执行
date: 2019-12-12 18:13:00
tags:
- Redis
---

# 协议格式
> The way RESP is used in Redis as a request-response protocol is the following:
> * Clients send commands to a Redis server as a RESP Array of Bulk Strings.
> * The server replies with one of the RESP types according to the command implementation.
>
> In RESP, the type of some data depends on the first byte:
> * For **Simple Strings** the first byte of the reply is `"+"`
> * For **Errors** the first byte of the reply is `"-"`
> * For **Integers** the first byte of the reply is `":"`
> * For **Bulk Strings** the first byte of the reply is `"$"`
> * For **Arrays** the first byte of the reply is `"*"`

引用自<https://redis.io/topics/protocol>。

* * *

# 建连与执行命令
Redis服务端通过使用`select`、`poll`等I/O多路复用系统调用来实现事件驱动模型。
Redis中的事件分为两类，一类被称为定时事件，如定期执行`serverCron`来处理过期逻辑、保存RDB、保存AOF等；另一类被称为文件事件，I/O多路复用函数在处理文件事件时使用，当与文件事件相关联的文件描述符ready即可读或可写时，`select`等函数返回，Redis使用与ready的文件描述符绑定的函数来处理相应的事件。
与文件描述符绑定的函数可分为三类：
1. AcceptHandler，用来`accept`客户端的连接，并且把`accept`后得到的文件描述符设置为非阻塞`O_NONBLOCK`
2. ReadHandler，用来`read`客户端发过来的数据，并且每当读到一条完整命令后就执行并把结果写到服务端的与该客户端相对应的缓冲区
3. WriteHandler，用来把服务端的与客户端对应的缓冲区中的数据`write`给客户端

![](https://img2018.cnblogs.com/blog/1224734/201912/1224734-20191213115837339-1247910637.jpg)

## ReadHandler
实现逻辑参照`src/networking.c:readQueryFromClient`。
每当`fd`可读，尝试读`REDIS_IOBUF_LEN=1024*16`这么多字节的数据到缓冲区，如果`nread=0`则断开与`fd`表示的客户端的链接。
当读到的数据不为空时，持续从缓冲区中尝试解析命令并执行，直到缓冲区已经没有完整命令。

![](https://img2018.cnblogs.com/blog/1224734/201912/1224734-20191213103813692-869512069.jpg)

## WriteHandler
实现逻辑参照`src/networking.c:sendReplyToClient`。
每当`fd`可写，循环把与`fd`对应的返回结果缓冲区中的全部数据都发给对应的客户端，直到全部发完或该次事件处理的`totwritten > REDIS_MAX_WRITE_PER_EVENT=1024*64`或`write`调用出错。
当`write`调用出错时，若错误`EAGAIN`为，则下一轮事件循环再尝试写这个`fd`；若为其他错误，则关闭与`fd`对应的客户端连接。

![](https://img2018.cnblogs.com/blog/1224734/201912/1224734-20191213104550236-230560978.jpg)

* * *

# Pipeline
>A Request/Response server can be implemented so that it is able to process new requests even if the client didn't already read the old responses. This way it is possible to send multiple commands to the server without waiting for the replies at all, and finally read the replies in a single step.

>This is called pipelining, and is a technique widely in use since many decades. For instance many POP3 protocol implementations already supported this feature, dramatically speeding up the process of downloading new emails from the server.

>Redis has supported pipelining since the very early days, so whatever version you are running, you can use pipelining with Redis. This is an example using the raw netcat utility:

```
$ (printf "PING\r\nPING\r\nPING\r\n"; sleep 1) | nc localhost 6379
+PONG
+PONG
+PONG
```
引用自<https://redis.io/topics/pipelining#redis-pipelining>。

按照官方文档，所谓pipeline只是客户端一起把多条命令发给服务端，然后一起读返回结果，而不是每发一条命令读一次返回结果。
在Redis服务端代码中并没有对所谓pipeline的特殊处理，服务端也只是每当能解析出一条命令就执行，然后把返回结果写到缓冲区里，然后在下次事件循环时按照`WriteHandler`中的逻辑把返回结果`write`给客户端。

## 命令行实现pipeline调用
按照官方文档，下面的命令即实现了一次pipeline调用
```
$ (printf '*2\r\n$3\r\nget\r\n$5\r\nhello\r\n*3\r\n$3\r\nset\r\n$5\r\nhello\r\n$5\r\nworld\r\n*2\r\n$3\r\nget\r\n$5\r\nhello\r\n*3\r\n$3\r\nset\r\n$8\r\n1 plus 1\r\n$1\r\n2\r\n*2\r\n$3\r\nget\r\n$8\r\n1 plus 1\r\n'; sleep 1) | nc localhost 6379
$-1
+OK
$5
world
+OK
$1
2
```
修改Redis代码加一些日志后，Redis服务端的输出为：
```
Poll and handle: START
Fd: 5 is readable
13021:M 12 Dec 17:44:44.657 - Accepted 127.0.0.1:57151
Create file event on fd: 6, mask: readable
Poll and handle: END
Poll and handle: START
Fd: 6 is readable
13021:M 12 Dec 17:44:44.657 - readQueryFromClient fd: 6, nread: 144
Create file event on fd: 6, mask: writeable
Poll and handle: END
Poll and handle: START
Fd: 6 is writeable
13021:M 12 Dec 17:44:44.658 - try sendReplyToClient to fd: 6
Delete file event on fd: 6, mask: writeable
Poll and handle: END
Poll and handle: START
Fd: 6 is readable
13021:M 12 Dec 17:44:45.638 - readQueryFromClient fd: 6, nread: 0
13021:M 12 Dec 17:44:45.638 - Client closed connection
Delete file event on fd: 6, mask: readable
Poll and handle: END
```
Wireshark抓包的结果：
![request](https://img2018.cnblogs.com/blog/1224734/201912/1224734-20191212181224838-519616019.png)
![response](https://img2018.cnblogs.com/blog/1224734/201912/1224734-20191212181243115-1484167716.png)

## Python实现pipeline调用
```python
# coding: utf-8

import redis


def main():
    client = redis.Redis(host='localhost', port=6379)
    pipe = client.pipeline(transaction=False)
    for i in range(16 * 1024):
        pipe.set(i, i)
    pipe.execute()


if __name__ == '__main__':
    main()

```
Redis日志：
```
Poll and handle: START
Fd: 5 is readable
13021:M 12 Dec 17:47:02.791 - Accepted 127.0.0.1:57168
Create file event on fd: 6, mask: readable
Poll and handle: END
Poll and handle: START
Fd: 6 is readable
13021:M 12 Dec 17:47:03.001 - readQueryFromClient fd: 6, nread: 16384
Create file event on fd: 6, mask: writeable
Poll and handle: END
Poll and handle: START
Fd: 6 is readable
13021:M 12 Dec 17:47:03.003 - readQueryFromClient fd: 6, nread: 16384
Fd: 6 is writeable
13021:M 12 Dec 17:47:03.004 - try sendReplyToClient to fd: 6
Delete file event on fd: 6, mask: writeable
Poll and handle: END
Poll and handle: START
Fd: 6 is readable
13021:M 12 Dec 17:47:03.004 - readQueryFromClient fd: 6, nread: 16384
Create file event on fd: 6, mask: writeable
Poll and handle: END
Poll and handle: START
Fd: 6 is readable
13021:M 12 Dec 17:47:03.005 - readQueryFromClient fd: 6, nread: 16384
Fd: 6 is writeable
13021:M 12 Dec 17:47:03.005 - try sendReplyToClient to fd: 6
Delete file event on fd: 6, mask: writeable
Poll and handle: END
Poll and handle: START
Fd: 6 is readable
13021:M 12 Dec 17:47:03.006 - readQueryFromClient fd: 6, nread: 16384
Create file event on fd: 6, mask: writeable
Poll and handle: END
Poll and handle: START
Fd: 6 is readable
13021:M 12 Dec 17:47:03.007 - readQueryFromClient fd: 6, nread: 16384
Fd: 6 is writeable
13021:M 12 Dec 17:47:03.008 - try sendReplyToClient to fd: 6
Delete file event on fd: 6, mask: writeable
Poll and handle: END
Poll and handle: START
Fd: 6 is readable
13021:M 12 Dec 17:47:03.008 - readQueryFromClient fd: 6, nread: 16384
Create file event on fd: 6, mask: writeable
Poll and handle: END
Poll and handle: START
Fd: 6 is readable
13021:M 12 Dec 17:47:03.010 - readQueryFromClient fd: 6, nread: 16384
Fd: 6 is writeable
13021:M 12 Dec 17:47:03.011 - try sendReplyToClient to fd: 6
Delete file event on fd: 6, mask: writeable
Poll and handle: END
Poll and handle: START
Fd: 6 is readable
13021:M 12 Dec 17:47:03.011 - readQueryFromClient fd: 6, nread: 16384
Create file event on fd: 6, mask: writeable
Poll and handle: END
Poll and handle: START
Fd: 6 is readable
13021:M 12 Dec 17:47:03.012 - readQueryFromClient fd: 6, nread: 16384
Fd: 6 is writeable
13021:M 12 Dec 17:47:03.013 - try sendReplyToClient to fd: 6
Delete file event on fd: 6, mask: writeable
Poll and handle: END
Poll and handle: START
Fd: 6 is readable
13021:M 12 Dec 17:47:03.013 - readQueryFromClient fd: 6, nread: 16384
Create file event on fd: 6, mask: writeable
Poll and handle: END
Poll and handle: START
Fd: 6 is readable
13021:M 12 Dec 17:47:03.014 - readQueryFromClient fd: 6, nread: 16384
Fd: 6 is writeable
13021:M 12 Dec 17:47:03.015 - try sendReplyToClient to fd: 6
Delete file event on fd: 6, mask: writeable
Poll and handle: END
Poll and handle: START
Fd: 6 is readable
13021:M 12 Dec 17:47:03.016 - readQueryFromClient fd: 6, nread: 16384
Create file event on fd: 6, mask: writeable
Poll and handle: END
Poll and handle: START
Fd: 6 is readable
13021:M 12 Dec 17:47:03.017 - readQueryFromClient fd: 6, nread: 16384
Fd: 6 is writeable
13021:M 12 Dec 17:47:03.017 - try sendReplyToClient to fd: 6
Delete file event on fd: 6, mask: writeable
Poll and handle: END
Poll and handle: START
Fd: 6 is readable
13021:M 12 Dec 17:47:03.018 - readQueryFromClient fd: 6, nread: 16384
Create file event on fd: 6, mask: writeable
Poll and handle: END
Poll and handle: START
Fd: 6 is readable
13021:M 12 Dec 17:47:03.018 - readQueryFromClient fd: 6, nread: 16384
Fd: 6 is writeable
13021:M 12 Dec 17:47:03.019 - try sendReplyToClient to fd: 6
Delete file event on fd: 6, mask: writeable
Poll and handle: END
Poll and handle: START
Fd: 6 is readable
13021:M 12 Dec 17:47:03.019 - readQueryFromClient fd: 6, nread: 16384
Create file event on fd: 6, mask: writeable
Poll and handle: END
Poll and handle: START
Fd: 6 is readable
13021:M 12 Dec 17:47:03.021 - readQueryFromClient fd: 6, nread: 16384
Fd: 6 is writeable
13021:M 12 Dec 17:47:03.022 - try sendReplyToClient to fd: 6
Delete file event on fd: 6, mask: writeable
Poll and handle: END
Poll and handle: START
Fd: 6 is readable
13021:M 12 Dec 17:47:03.022 - readQueryFromClient fd: 6, nread: 16384
Create file event on fd: 6, mask: writeable
Poll and handle: END
Poll and handle: START
Fd: 6 is readable
13021:M 12 Dec 17:47:03.024 - readQueryFromClient fd: 6, nread: 16384
Fd: 6 is writeable
13021:M 12 Dec 17:47:03.025 - try sendReplyToClient to fd: 6
Delete file event on fd: 6, mask: writeable
Poll and handle: END
Poll and handle: START
Fd: 6 is readable
13021:M 12 Dec 17:47:03.025 - readQueryFromClient fd: 6, nread: 16384
Create file event on fd: 6, mask: writeable
Poll and handle: END
Poll and handle: START
Fd: 6 is readable
13021:M 12 Dec 17:47:03.026 - readQueryFromClient fd: 6, nread: 16384
Fd: 6 is writeable
13021:M 12 Dec 17:47:03.028 - try sendReplyToClient to fd: 6
Delete file event on fd: 6, mask: writeable
Poll and handle: END
Poll and handle: START
Fd: 6 is readable
13021:M 12 Dec 17:47:03.028 - readQueryFromClient fd: 6, nread: 16384
Create file event on fd: 6, mask: writeable
Poll and handle: END
Poll and handle: START
Fd: 6 is readable
13021:M 12 Dec 17:47:03.029 - readQueryFromClient fd: 6, nread: 16384
Fd: 6 is writeable
13021:M 12 Dec 17:47:03.033 - try sendReplyToClient to fd: 6
Delete file event on fd: 6, mask: writeable
Poll and handle: END
Poll and handle: START
Fd: 6 is readable
13021:M 12 Dec 17:47:03.033 - readQueryFromClient fd: 6, nread: 16384
Create file event on fd: 6, mask: writeable
Poll and handle: END
Poll and handle: START
Fd: 6 is readable
13021:M 12 Dec 17:47:03.034 - readQueryFromClient fd: 6, nread: 16384
Fd: 6 is writeable
13021:M 12 Dec 17:47:03.035 - try sendReplyToClient to fd: 6
Delete file event on fd: 6, mask: writeable
Poll and handle: END
Poll and handle: START
Fd: 6 is readable
13021:M 12 Dec 17:47:03.035 - readQueryFromClient fd: 6, nread: 16384
Create file event on fd: 6, mask: writeable
Poll and handle: END
Poll and handle: START
Fd: 6 is readable
13021:M 12 Dec 17:47:03.037 - readQueryFromClient fd: 6, nread: 16384
Fd: 6 is writeable
13021:M 12 Dec 17:47:03.038 - try sendReplyToClient to fd: 6
Delete file event on fd: 6, mask: writeable
Poll and handle: END
Poll and handle: START
Fd: 6 is readable
13021:M 12 Dec 17:47:03.038 - readQueryFromClient fd: 6, nread: 16384
Create file event on fd: 6, mask: writeable
Poll and handle: END
Poll and handle: START
Fd: 6 is readable
13021:M 12 Dec 17:47:03.039 - readQueryFromClient fd: 6, nread: 16384
Fd: 6 is writeable
13021:M 12 Dec 17:47:03.040 - try sendReplyToClient to fd: 6
Delete file event on fd: 6, mask: writeable
Poll and handle: END
Poll and handle: START
Fd: 6 is readable
13021:M 12 Dec 17:47:03.040 - readQueryFromClient fd: 6, nread: 16384
Create file event on fd: 6, mask: writeable
Poll and handle: END
Poll and handle: START
Fd: 6 is readable
13021:M 12 Dec 17:47:03.042 - readQueryFromClient fd: 6, nread: 16384
Fd: 6 is writeable
13021:M 12 Dec 17:47:03.042 - try sendReplyToClient to fd: 6
Delete file event on fd: 6, mask: writeable
Poll and handle: END
Poll and handle: START
Fd: 6 is readable
13021:M 12 Dec 17:47:03.042 - readQueryFromClient fd: 6, nread: 16384
Create file event on fd: 6, mask: writeable
Poll and handle: END
Poll and handle: START
Fd: 6 is readable
13021:M 12 Dec 17:47:03.043 - readQueryFromClient fd: 6, nread: 10548
Fd: 6 is writeable
13021:M 12 Dec 17:47:03.044 - try sendReplyToClient to fd: 6
Delete file event on fd: 6, mask: writeable
Poll and handle: END
Poll and handle: START
Fd: 6 is readable
13021:M 12 Dec 17:47:03.108 - readQueryFromClient fd: 6, nread: 0
13021:M 12 Dec 17:47:03.108 - Client closed connection
Delete file event on fd: 6, mask: readable
Poll and handle: END
```

## 包含大量命令的pipeline
把上面python代码`range`的参数改为`16 * 1024 * 100`，出现了`13429:M 12 Dec 18:00:18.557 - sendReplyToClient EAGAIN, fd: 6`这种日志。

* * *

# 多个客户端同时执行Pipeline会发生什么
`range`参数改为`16 * 1024 * 10`，命令行执行`python main.py & python main.py`。
Redis日志：
```
Poll and handle: START
Fd: 5 is readable
13596:M 12 Dec 18:05:49.267 - Accepted 127.0.0.1:57368
Create file event on fd: 6, mask: readable
Poll and handle: END
Poll and handle: START
Fd: 5 is readable
13596:M 12 Dec 18:05:49.269 - Accepted 127.0.0.1:57369
Create file event on fd: 7, mask: readable
Poll and handle: END
13596:M 12 Dec 18:05:53.103 - 2 clients connected (0 slaves), 993008 bytes in use
13596:M 12 Dec 18:05:58.237 - 2 clients connected (0 slaves), 993008 bytes in use
13596:M 12 Dec 18:06:03.371 - 2 clients connected (0 slaves), 993008 bytes in use
13596:M 12 Dec 18:06:08.513 - 2 clients connected (0 slaves), 993008 bytes in use
Poll and handle: START
Fd: 6 is readable
13596:M 12 Dec 18:06:09.131 - readQueryFromClient fd: 6, nread: 16384
Create file event on fd: 6, mask: writeable
Poll and handle: END
Poll and handle: START
Fd: 6 is readable
13596:M 12 Dec 18:06:09.134 - readQueryFromClient fd: 6, nread: 16384
Fd: 6 is writeable
13596:M 12 Dec 18:06:09.134 - try sendReplyToClient to fd: 6
Delete file event on fd: 6, mask: writeable
Poll and handle: END
Poll and handle: START
Fd: 6 is readable
13596:M 12 Dec 18:06:09.134 - readQueryFromClient fd: 6, nread: 16384
Create file event on fd: 6, mask: writeable
Poll and handle: END
Poll and handle: START
Fd: 6 is readable
13596:M 12 Dec 18:06:09.135 - readQueryFromClient fd: 6, nread: 16384
Fd: 6 is writeable
13596:M 12 Dec 18:06:09.136 - try sendReplyToClient to fd: 6
Delete file event on fd: 6, mask: writeable
Poll and handle: END
................................
Poll and handle: START
Fd: 6 is readable
13596:M 12 Dec 18:06:09.283 - readQueryFromClient fd: 6, nread: 16384
Fd: 7 is readable
13596:M 12 Dec 18:06:09.284 - readQueryFromClient fd: 7, nread: 16384
Fd: 6 is writeable
13596:M 12 Dec 18:06:09.285 - try sendReplyToClient to fd: 6
Delete file event on fd: 6, mask: writeable
Fd: 7 is writeable
13596:M 12 Dec 18:06:09.285 - try sendReplyToClient to fd: 7
Delete file event on fd: 7, mask: writeable
Poll and handle: END
Poll and handle: START
Fd: 6 is readable
13596:M 12 Dec 18:06:09.285 - readQueryFromClient fd: 6, nread: 16384
Create file event on fd: 6, mask: writeable
Fd: 7 is readable
13596:M 12 Dec 18:06:09.285 - readQueryFromClient fd: 7, nread: 16384
Create file event on fd: 7, mask: writeable
Poll and handle: END
Poll and handle: START
Fd: 6 is readable
13596:M 12 Dec 18:06:09.286 - readQueryFromClient fd: 6, nread: 16384
Fd: 7 is readable
13596:M 12 Dec 18:06:09.287 - readQueryFromClient fd: 7, nread: 16384
Fd: 6 is writeable
13596:M 12 Dec 18:06:09.287 - try sendReplyToClient to fd: 6
Delete file event on fd: 6, mask: writeable
Fd: 7 is writeable
13596:M 12 Dec 18:06:09.287 - try sendReplyToClient to fd: 7
Delete file event on fd: 7, mask: writeable
Poll and handle: END
Poll and handle: START
Fd: 6 is readable
13596:M 12 Dec 18:06:09.288 - readQueryFromClient fd: 6, nread: 16384
Create file event on fd: 6, mask: writeable
Fd: 7 is readable
13596:M 12 Dec 18:06:09.288 - readQueryFromClient fd: 7, nread: 16384
Create file event on fd: 7, mask: writeable
Poll and handle: END
Poll and handle: START
Fd: 6 is readable
13596:M 12 Dec 18:06:09.289 - readQueryFromClient fd: 6, nread: 16384
Fd: 7 is readable
13596:M 12 Dec 18:06:09.290 - readQueryFromClient fd: 7, nread: 16384
Fd: 6 is writeable
13596:M 12 Dec 18:06:09.291 - try sendReplyToClient to fd: 6
Delete file event on fd: 6, mask: writeable
Fd: 7 is writeable
13596:M 12 Dec 18:06:09.291 - try sendReplyToClient to fd: 7
Delete file event on fd: 7, mask: writeable
Poll and handle: END
................................
Poll and handle: START
Fd: 6 is writeable
13596:M 12 Dec 18:06:19.984 - try sendReplyToClient to fd: 6
13596:M 12 Dec 18:06:19.984 - sendReplyToClient EAGAIN, fd: 6
Poll and handle: END
Poll and handle: START
Fd: 7 is writeable
13596:M 12 Dec 18:06:20.121 - try sendReplyToClient to fd: 7
Poll and handle: END
Poll and handle: START
Fd: 7 is writeable
13596:M 12 Dec 18:06:20.122 - try sendReplyToClient to fd: 7
Poll and handle: END
Poll and handle: START
Fd: 7 is writeable
13596:M 12 Dec 18:06:20.122 - try sendReplyToClient to fd: 7
Poll and handle: END
Poll and handle: START
Fd: 7 is writeable
13596:M 12 Dec 18:06:20.122 - try sendReplyToClient to fd: 7
13596:M 12 Dec 18:06:20.122 - sendReplyToClient EAGAIN, fd: 7
Poll and handle: END
................................
Poll and handle: START
Fd: 7 is writeable
13596:M 12 Dec 18:06:23.854 - try sendReplyToClient to fd: 7
13596:M 12 Dec 18:06:23.854 - sendReplyToClient EAGAIN, fd: 7
Poll and handle: END
Poll and handle: START
Fd: 6 is writeable
13596:M 12 Dec 18:06:24.085 - try sendReplyToClient to fd: 6
Poll and handle: END
Poll and handle: START
Fd: 6 is writeable
13596:M 12 Dec 18:06:24.085 - try sendReplyToClient to fd: 6
Delete file event on fd: 6, mask: writeable
Poll and handle: END
Poll and handle: START
Fd: 7 is writeable
13596:M 12 Dec 18:06:24.192 - try sendReplyToClient to fd: 7
Poll and handle: END
Poll and handle: START
Fd: 7 is writeable
13596:M 12 Dec 18:06:24.192 - try sendReplyToClient to fd: 7
Delete file event on fd: 7, mask: writeable
Poll and handle: END
Poll and handle: START
Fd: 6 is readable
13596:M 12 Dec 18:06:26.373 - readQueryFromClient fd: 6, nread: 0
13596:M 12 Dec 18:06:26.373 - Client closed connection
Delete file event on fd: 6, mask: readable
Poll and handle: END
Poll and handle: START
Fd: 7 is readable
13596:M 12 Dec 18:06:26.510 - readQueryFromClient fd: 7, nread: 0
13596:M 12 Dec 18:06:26.510 - Client closed connection
Delete file event on fd: 7, mask: readable
Poll and handle: END
```