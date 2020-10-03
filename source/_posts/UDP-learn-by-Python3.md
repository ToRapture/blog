---
title: UDP learn by Python3
date: 2020-04-27 18:30:00
tags: Network, TCP/IP
---

## Find max packet size
```python
# coding: utf-8

import socket


def find_max_udp_packet_size():
    sock = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
    max_sz = 0
    l, r = 0, 65535 - 8 - 20
    while l <= r:
        mid = int((l + r) / 2)
        ok = True

        try:
            sock.sendto(b'A' * mid, ('localhost', 9))
        except socket.error:
            ok = False

        if ok:
            max_sz = mid
            l = mid + 1
        else:
            r = mid - 1

    return max_sz


def main():
    max_sz = find_max_udp_packet_size()
    print('max_sz = %s' % max_sz)


if __name__ == '__main__':
    main()

```
I used the code above to find the max size of a UDP packet size by default configuration.
The result on my computer is:
```
$ uname -v
Darwin Kernel Version 19.2.0: Sat Nov  9 03:47:04 PST 2019; root:xnu-6153.61.1~20/RELEASE_X86_64
$ python3 --version
Python 3.6.4
$ python3 udp_packet_size.py
max_sz = 9216
```

## More test
Use the code below to test.
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

```
In [1]: sock.bind(('localhost', 10001))

In [2]: receiver = ('localhost', 10002)

In [3]: sock.sendto(b'01234567', receiver)
Out[3]: 8

In [4]: sock.sendto(b'01234567', receiver)
Out[4]: 8

In [5]: sock.sendto(b'01234567', receiver)
Out[5]: 8

In [6]: sock.sendto(b'01234567', receiver)
Out[6]: 8
```

```

In [1]: sock.bind(('localhost', 10002))

In [2]: receiver = ('localhost', 10001)

In [3]: print(sock.recvfrom(1024))
(b'01234567', ('127.0.0.1', 10001))

In [4]: print(sock.recvfrom(4))
(b'0123', ('127.0.0.1', 10001))

In [5]: print(sock.recvfrom(4))
(b'0123', ('127.0.0.1', 10001))

In [6]: print(sock.recvfrom(4))
(b'0123', ('127.0.0.1', 10001))

In [7]: print(sock.recvfrom(4))
```
We can see that if the `bufsize` is smaller the length of the packet received, will only read `bufsize` bytes of the received data, and the rest of that packet will not be returned in subsequent reads.