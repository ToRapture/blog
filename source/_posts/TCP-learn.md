---
title: TCP learn
date: 2020-05-09 17:48:00
tags:
- Network
- TCP/IP
---

# Headers
![](https://img2020.cnblogs.com/blog/1224734/202005/1224734-20200509174526548-2078724920.png)

# Connection Management

## State Tansition Diagram
![](https://img2020.cnblogs.com/blog/1224734/202005/1224734-20200509174539501-1041931082.png)

## Sequence Diagram
### Normal
![](https://img2020.cnblogs.com/blog/1224734/202005/1224734-20200509174607910-138499393.png)
### Simultaneous Open
![](https://img2020.cnblogs.com/blog/1224734/202005/1224734-20200509174619105-1272712747.png)
### Simultaneous Close
![](https://img2020.cnblogs.com/blog/1224734/202005/1224734-20200509174626926-1002849977.png)
### Half Close
![](https://img2020.cnblogs.com/blog/1224734/202005/1224734-20200509174634166-1111036533.png)
### Normal with State
![](https://img2020.cnblogs.com/blog/1224734/202005/1224734-20200509175009901-1461619215.png)

# Examples

`server`:
```
In [1]: import socket

In [2]: s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)

In [3]: s.bind(('localhost', 30002))

In [4]: s.listen(5)

In [5]: conn, addr = s.accept()

In [6]: conn.send(b'who are you')
Out[6]: 11

In [7]: conn.recv(1024)
Out[7]: b'hello world'

In [8]: conn.recv(1024)
Out[8]: b''

In [9]: conn.close()
```

`client`
```
In [1]: import socket

In [2]: s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)

In [3]: s.connect(('localhost', 30002))

In [4]: s.send(b'hello world')
Out[4]: 11

In [5]: s.recv(1024)
Out[5]: b'who are you'

In [6]: s.close()
```

`netstat`
```
$ netstat -tan | grep 30002
tcp        0      0 127.0.0.1:30002         0.0.0.0:*               LISTEN
$ netstat -tan | grep 30002
tcp        0      0 127.0.0.1:30002         0.0.0.0:*               LISTEN
tcp        0      0 127.0.0.1:30002         127.0.0.1:48496         ESTABLISHED
tcp        0      0 127.0.0.1:48496         127.0.0.1:30002         ESTABLISHED
$ netstat -tan | grep 30002
tcp        0      0 127.0.0.1:30002         0.0.0.0:*               LISTEN
tcp        0      0 127.0.0.1:30002         127.0.0.1:48496         CLOSE_WAIT
tcp        0      0 127.0.0.1:48496         127.0.0.1:30002         FIN_WAIT2
$ netstat -tan | grep 30002
tcp        0      0 127.0.0.1:30002         0.0.0.0:*               LISTEN
tcp        0      0 127.0.0.1:48496         127.0.0.1:30002         TIME_WAIT
```

`tcpdump`
```
$ sudo tcpdump -ilo "tcp and port 30002" -v -X
tcpdump: listening on lo, link-type EN10MB (Ethernet), capture size 262144 bytes
17:41:56.089133 IP (tos 0x0, ttl 64, id 29424, offset 0, flags [DF], proto TCP (6), length 60)
    localhost.48496 > localhost.30002: Flags [S], cksum 0xfe30 (incorrect -> 0xc989), seq 310491251, win 43690, options [mss 65495,sackOK,TS val 327862915 ecr 0,nop,wscale 7], length 0
	0x0000:  4500 003c 72f0 4000 4006 c9c9 7f00 0001  E..<r.@.@.......
	0x0010:  7f00 0001 bd70 7532 1281 b873 0000 0000  .....pu2...s....
	0x0020:  a002 aaaa fe30 0000 0204 ffd7 0402 080a  .....0..........
	0x0030:  138a ca83 0000 0000 0103 0307            ............
17:41:56.089145 IP (tos 0x0, ttl 64, id 0, offset 0, flags [DF], proto TCP (6), length 60)
    localhost.30002 > localhost.48496: Flags [S.], cksum 0xfe30 (incorrect -> 0x8343), seq 33121838, ack 310491252, win 43690, options [mss 65495,sackOK,TS val 327862915 ecr 327862915,nop,wscale 7], length 0
	0x0000:  4500 003c 0000 4000 4006 3cba 7f00 0001  E..<..@.@.<.....
	0x0010:  7f00 0001 7532 bd70 01f9 662e 1281 b874  ....u2.p..f....t
	0x0020:  a012 aaaa fe30 0000 0204 ffd7 0402 080a  .....0..........
	0x0030:  138a ca83 138a ca83 0103 0307            ............
17:41:56.089155 IP (tos 0x0, ttl 64, id 29425, offset 0, flags [DF], proto TCP (6), length 52)
    localhost.48496 > localhost.30002: Flags [.], cksum 0xfe28 (incorrect -> 0x5588), ack 1, win 342, options [nop,nop,TS val 327862915 ecr 327862915], length 0
	0x0000:  4500 0034 72f1 4000 4006 c9d0 7f00 0001  E..4r.@.@.......
	0x0010:  7f00 0001 bd70 7532 1281 b874 01f9 662f  .....pu2...t..f/
	0x0020:  8010 0156 fe28 0000 0101 080a 138a ca83  ...V.(..........
	0x0030:  138a ca83                                ....
17:42:16.978059 IP (tos 0x0, ttl 64, id 29426, offset 0, flags [DF], proto TCP (6), length 63)
    localhost.48496 > localhost.30002: Flags [P.], cksum 0xfe33 (incorrect -> 0xaf40), seq 1:12, ack 1, win 342, options [nop,nop,TS val 327868137 ecr 327862915], length 11
	0x0000:  4500 003f 72f2 4000 4006 c9c4 7f00 0001  E..?r.@.@.......
	0x0010:  7f00 0001 bd70 7532 1281 b874 01f9 662f  .....pu2...t..f/
	0x0020:  8018 0156 fe33 0000 0101 080a 138a dee9  ...V.3..........
	0x0030:  138a ca83 6865 6c6c 6f20 776f 726c 64    ....hello.world
17:42:16.978068 IP (tos 0x0, ttl 64, id 44204, offset 0, flags [DF], proto TCP (6), length 52)
    localhost.30002 > localhost.48496: Flags [.], cksum 0xfe28 (incorrect -> 0x2cb1), ack 12, win 342, options [nop,nop,TS val 327868137 ecr 327868137], length 0
	0x0000:  4500 0034 acac 4000 4006 9015 7f00 0001  E..4..@.@.......
	0x0010:  7f00 0001 7532 bd70 01f9 662f 1281 b87f  ....u2.p..f/....
	0x0020:  8010 0156 fe28 0000 0101 080a 138a dee9  ...V.(..........
	0x0030:  138a dee9                                ....
17:42:26.921826 IP (tos 0x0, ttl 64, id 44205, offset 0, flags [DF], proto TCP (6), length 63)
    localhost.30002 > localhost.48496: Flags [P.], cksum 0xfe33 (incorrect -> 0x875c), seq 1:12, ack 12, win 342, options [nop,nop,TS val 327870623 ecr 327868137], length 11
	0x0000:  4500 003f acad 4000 4006 9009 7f00 0001  E..?..@.@.......
	0x0010:  7f00 0001 7532 bd70 01f9 662f 1281 b87f  ....u2.p..f/....
	0x0020:  8018 0156 fe33 0000 0101 080a 138a e89f  ...V.3..........
	0x0030:  138a dee9 7768 6f20 6172 6520 796f 75    ....who.are.you
17:42:26.921835 IP (tos 0x0, ttl 64, id 29427, offset 0, flags [DF], proto TCP (6), length 52)
    localhost.48496 > localhost.30002: Flags [.], cksum 0xfe28 (incorrect -> 0x193a), ack 12, win 342, options [nop,nop,TS val 327870623 ecr 327870623], length 0
	0x0000:  4500 0034 72f3 4000 4006 c9ce 7f00 0001  E..4r.@.@.......
	0x0010:  7f00 0001 bd70 7532 1281 b87f 01f9 663a  .....pu2......f:
	0x0020:  8010 0156 fe28 0000 0101 080a 138a e89f  ...V.(..........
	0x0030:  138a e89f                                ....
17:42:39.804551 IP (tos 0x0, ttl 64, id 29428, offset 0, flags [DF], proto TCP (6), length 52)
    localhost.48496 > localhost.30002: Flags [F.], cksum 0xfe28 (incorrect -> 0x0ca4), seq 12, ack 12, win 342, options [nop,nop,TS val 327873844 ecr 327870623], length 0
	0x0000:  4500 0034 72f4 4000 4006 c9cd 7f00 0001  E..4r.@.@.......
	0x0010:  7f00 0001 bd70 7532 1281 b87f 01f9 663a  .....pu2......f:
	0x0020:  8011 0156 fe28 0000 0101 080a 138a f534  ...V.(.........4
	0x0030:  138a e89f                                ....
17:42:39.843347 IP (tos 0x0, ttl 64, id 44206, offset 0, flags [DF], proto TCP (6), length 52)
    localhost.30002 > localhost.48496: Flags [.], cksum 0xfe28 (incorrect -> 0x0005), ack 13, win 342, options [nop,nop,TS val 327873854 ecr 327873844], length 0
	0x0000:  4500 0034 acae 4000 4006 9013 7f00 0001  E..4..@.@.......
	0x0010:  7f00 0001 7532 bd70 01f9 663a 1281 b880  ....u2.p..f:....
	0x0020:  8010 0156 fe28 0000 0101 080a 138a f53e  ...V.(.........>
	0x0030:  138a f534                                ...4
17:42:51.138882 IP (tos 0x0, ttl 64, id 44207, offset 0, flags [DF], proto TCP (6), length 52)
    localhost.30002 > localhost.48496: Flags [F.], cksum 0xfe28 (incorrect -> 0xf4fc), seq 12, ack 13, win 342, options [nop,nop,TS val 327876677 ecr 327873844], length 0
	0x0000:  4500 0034 acaf 4000 4006 9012 7f00 0001  E..4..@.@.......
	0x0010:  7f00 0001 7532 bd70 01f9 663a 1281 b880  ....u2.p..f:....
	0x0020:  8011 0156 fe28 0000 0101 080a 138b 0045  ...V.(.........E
	0x0030:  138a f534                                ...4
17:42:51.138897 IP (tos 0x0, ttl 64, id 27317, offset 0, flags [DF], proto TCP (6), length 52)
    localhost.48496 > localhost.30002: Flags [.], cksum 0xe9eb (correct), ack 13, win 342, options [nop,nop,TS val 327876677 ecr 327876677], length 0
	0x0000:  4500 0034 6ab5 4000 4006 d20c 7f00 0001  E..4j.@.@.......
	0x0010:  7f00 0001 bd70 7532 1281 b880 01f9 663b  .....pu2......f;
	0x0020:  8010 0156 e9eb 0000 0101 080a 138b 0045  ...V...........E
	0x0030:  138b 0045                                ...E
```