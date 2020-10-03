---
title: UDP and netstat
date: 2020-04-28 12:49:00
tags: Network, TCP/IP
---

Do some experiment about UDP by Python3 and netstat.

## Code
```
# coding: utf-8

import socket

from IPython import embed


def main():
    sock = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
    embed()
    sock.close()


if __name__ == '__main__':
    main()

```

### A and B start
A:
```
$ python3 client.py
Python 3.5.2 (default, Oct  8 2019, 13:06:37)
Type "copyright", "credits" or "license" for more information.

IPython 2.4.1 -- An enhanced Interactive Python.
?         -> Introduction and overview of IPython's features.
%quickref -> Quick reference.
help      -> Python's own help system.
object?   -> Details about 'object', use 'object??' for extra details.

In [1]:
```

B:
```
$ python3 client.py
Python 3.5.2 (default, Oct  8 2019, 13:06:37)
Type "copyright", "credits" or "license" for more information.

IPython 2.4.1 -- An enhanced Interactive Python.
?         -> Introduction and overview of IPython's features.
%quickref -> Quick reference.
help      -> Python's own help system.
object?   -> Details about 'object', use 'object??' for extra details.

In [1]:
```
```
$ netstat -au
Active Internet connections (servers and established)
Proto Recv-Q Send-Q Local Address           Foreign Address         State
```

### A bind
A:
```
In [1]: sock.bind(('localhost', 10001))
```
```
$ netstat -au
Active Internet connections (servers and established)
Proto Recv-Q Send-Q Local Address           Foreign Address         State
udp        0      0 localhost:10001         *:*
```

### B send to A
B:
```
In [1]: sock.sendto(b'hello world', ('localhost', 10001))
Out[1]: 11
```

A:
```
In [2]: sock.recvfrom(1024)
Out[2]: (b'hello world', ('127.0.0.1', 42616))
```

```
$ netstat -au
Active Internet connections (servers and established)
Proto Recv-Q Send-Q Local Address           Foreign Address         State
udp      768      0 localhost:10001         *:*
udp        0      0 *:42616                 *:*
```

### A send to B
A:
```
In [3]: sock.sendto(b'This is a response.', ('127.0.0.1', 42616))
Out[3]: 19
```

B:
```
In [2]: sock.recvfrom(1024)
Out[2]: (b'This is a response.', ('127.0.0.1', 10001))
```

```
$ netstat -au
Active Internet connections (servers and established)
Proto Recv-Q Send-Q Local Address           Foreign Address         State
udp        0      0 localhost:10001         *:*
udp        0      0 *:42616                 *:*
```

### B connect to A
B:
```
In [4]: sock.connect(('127.0.0.1', 10001))
```

```
$ netstat -au
Active Internet connections (servers and established)
Proto Recv-Q Send-Q Local Address           Foreign Address         State
udp        0      0 localhost:10001         *:*
udp        0      0 localhost:42616         localhost:10001         ESTABLISHED
```

A:
```
In [4]: sock.sendto(b'This is another response.', ('127.0.0.1', 42616))
Out[4]: 25
```

B:
```
In [5]: sock.recvfrom(1024)
Out[5]: (b'This is another response.', ('127.0.0.1', 10001))
```

C:
```
In [1]: sock.bind(('127.0.0.1', 10010))

In [2]: sock.sendto(b'This is a response from C.', ('127.0.0.1', 42616))
Out[2]: 26
```

B:
```
In [7]: sock.recvfrom(1024)
blocked
```
After B connected to A, B will not receive packets from any endpoint other than A.