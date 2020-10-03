---
title: UNIX编程 TCP基础读写笔记
date: 2019-12-04 15:30:00
mathjax: true
tags:
- UNIX编程
---

## 基本TCP客户端与服务器
### Server
```cpp
#include <cstdio>
#include <cstring>

#include <string>
#include <vector>

#include <unistd.h>
#include <arpa/inet.h>
#include <sys/errno.h>
#include <sys/socket.h>
#include <netinet/in.h>
#include <string>

using namespace std;

const int BUF_SIZE = 1024;

string addr_to_string(const sockaddr_in *addr) {
    char addr_str[INET_ADDRSTRLEN];
    const char *addr_str_ptr = inet_ntop(AF_INET, &addr->sin_addr, addr_str, sizeof(addr_str));
    if (addr_str_ptr == NULL) {
        fprintf(stderr, "inet_ntop failed\n");
        return "";
    }
    return string(addr_str_ptr) + ":" + to_string(ntohs(addr->sin_port));
}

string get_sockname_string(int sockfd) {
    sockaddr_in addr;
    socklen_t len = sizeof(addr);
    if (getsockname(sockfd, (sockaddr *) &addr, &len) == -1) {
        fprintf(stderr, "getsockname failed\n");
        return "";
    }
    return addr_to_string(&addr);
}

string get_peername_string(int sockfd) {
    sockaddr_in addr;
    socklen_t len = sizeof(addr);
    if (getpeername(sockfd, (sockaddr *) &addr, &len) == -1) {
        fprintf(stderr, "getpeername failed, errno: %d, str: %s\n", errno, strerror(errno));
        return "";
    }
    return addr_to_string(&addr);
}


int init_server(int type, const sockaddr *addr, socklen_t alen, int qlen) {
    int fd;
    if ((fd = socket(addr->sa_family, type, 0)) == -1) {
        fprintf(stderr, "create socket failed, errno: %d, str: %s\n", errno, strerror(errno));
        return -1;
    }
    printf("create socket fd: %d, sock_name: %s, peer_name: %s\n", fd, get_sockname_string(fd).c_str(), get_peername_string(fd).c_str());

    if (bind(fd, addr, alen) == -1) {
        fprintf(stderr, "bind failed, errno: %d, str: %s\n", errno, strerror(errno));
        close(fd);
        return -1;
    }
    printf("bind socket fd: %d, sock_name: %s, peer_name: %s\n", fd, get_sockname_string(fd).c_str(), get_peername_string(fd).c_str());

    if (type == SOCK_STREAM || type == SOCK_SEQPACKET) {
        if (listen(fd, qlen) == -1) {
            fprintf(stderr, "listen failed, errno: %d, str: %s\n", errno, strerror(errno));
            close(fd);
            return -1;
        }
        printf("listen fd: %d, sock_name: %s, peer_name: %s\n", fd, get_sockname_string(fd).c_str(), get_peername_string(fd).c_str());
    }
    return fd;
}

int main(int argc, char **argv) {
    int sleep_before_recv = 0;
    int sleep_after_listen = 0;
    int qlen = 0;
    int port = 27015;
    char *ip = "0.0.0.0";

    for (int i = 1; i < argc;) {
        if (argv[i] == string("--port")) {
            port = atoi(argv[i + 1]);
            i += 2;
            continue;
        }
        if (argv[i] == string("--qlen")) {
            qlen = atoi(argv[i + 1]);
            i += 2;
            continue;
        }
        if (argv[i] == string("--sleep-before-recv")) {
            sleep_before_recv = atoi(argv[i + 1]);
            i += 2;
            continue;
        }
        if (argv[i] == string("--sleep-after-listen")) {
            sleep_after_listen = atoi(argv[i + 1]);
            i += 2;
            continue;
        }
        if (argv[i] == string("--ip")) {
            ip = argv[i + 1];
            i += 2;
            continue;
        }
        i++;
    }

    printf("port: %d, qlen: %d, sleep-before-recv: %d, sleep-after-listen: %d\n", port, qlen, sleep_before_recv, sleep_after_listen);

    sockaddr_in addr;
    bzero(&addr, sizeof(addr)); // APUE 16.3.2 其中成员sin_zero为填充字段，应该全部被置为0
    addr.sin_family = AF_INET;
    inet_pton(addr.sin_family, ip, &addr.sin_addr);
    addr.sin_port = htons(port);

    int sockfd = init_server(SOCK_STREAM, (sockaddr *) &addr, sizeof(addr), qlen);
    if (sockfd == -1) return -1;

    sleep(sleep_after_listen);

    while (true) {
        int clfd, n;
        sockaddr_in accepted_addr;
        socklen_t len = sizeof(accepted_addr);

        if ((clfd = accept(sockfd, (sockaddr *) &accepted_addr, &len)) == -1) {
            fprintf(stderr, "accept failed, errno: %d, str: %s\n", errno, strerror(errno));
            continue;
        }
        printf("after accept sockfd: %d, sock_name: %s, peer_name: %s\n",
               sockfd, get_sockname_string(sockfd).c_str(), get_peername_string(sockfd).c_str());
        printf("accepted fd: %d, sock_name: %s, peer_name: %s, addr: %s\n",
               clfd, get_sockname_string(clfd).c_str(), get_peername_string(clfd).c_str(), addr_to_string(&accepted_addr).c_str());

        sleep(sleep_before_recv);
        char buf[BUF_SIZE];
        if ((n = recv(clfd, buf, BUF_SIZE - 1, 0)) == -1) {
            fprintf(stderr, "recv failed, errno: %d, str: %s\n", errno, strerror(errno));
            close(clfd);
            continue;
        }
        buf[n] = '\0';
        printf("read %d bytes from fd: %d, buf: %s\n", n, clfd, buf);
        close(clfd);
    }

    return 0;
}
```

如果调用`getsockname`时没有地址绑定到传入的套接字，则其结果是未定义的。[^getsockname]

### Client
```cpp
#include <cstdio>
#include <cstring>

#include <vector>
#include <string>

#include <unistd.h>
#include <arpa/inet.h>
#include <sys/errno.h>
#include <sys/socket.h>
#include <netinet/in.h>

using namespace std;

const int BUF_SIZE = 1024;

string addr_to_string(const sockaddr_in *addr) {
    char addr_str[INET_ADDRSTRLEN];
    const char *addr_str_ptr = inet_ntop(AF_INET, &addr->sin_addr, addr_str, sizeof(addr_str));
    if (addr_str_ptr == NULL) {
        fprintf(stderr, "inet_ntop failed\n");
        return "";
    }
    return string(addr_str_ptr) + ":" + to_string(ntohs(addr->sin_port));
}

string get_sockname_string(int sockfd) {
    sockaddr_in addr;
    socklen_t len = sizeof(addr);
    if (getsockname(sockfd, (sockaddr *) &addr, &len) == -1) {
        fprintf(stderr, "getsockname failed\n");
        return "";
    }
    return addr_to_string(&addr);
}

string get_peername_string(int sockfd) {
    sockaddr_in addr;
    socklen_t len = sizeof(addr);
    if (getpeername(sockfd, (sockaddr *) &addr, &len) == -1) {
        fprintf(stderr, "getpeername failed, errno: %d, str: %s\n", errno, strerror(errno));
        return "";
    }
    return addr_to_string(&addr);
}

int main(int argc, char **argv) {
    int port = 27015;
    char *ip = "0.0.0.0";
    for (int i = 1; i < argc;) {
        if (argv[i] == string("--port")) {
            port = atoi(argv[i + 1]);
            i += 2;
            continue;
        }
        if (argv[i] == string("--ip")) {
            ip = argv[i + 1];
            i += 2;
            continue;
        }
        i++;
    }


    sockaddr_in addr;
    bzero(&addr, sizeof(addr));
    addr.sin_family = AF_INET;
    addr.sin_port = htons(port);
    if (inet_pton(addr.sin_family, ip, &addr.sin_addr) <= 0) {
        fprintf(stderr, "inet_pton failed\n");
        return -1;
    }

    int sockfd = socket(addr.sin_family, SOCK_STREAM, 0);
    if (sockfd == -1) {
        fprintf(stderr, "create socket failed, errno: %d, str: %s\n", errno, strerror(errno));
        return -1;
    }
    printf("fd: %d, sock_name: %s, peer_name: %s\n", sockfd, get_sockname_string(sockfd).c_str(), get_peername_string(sockfd).c_str());

    if (connect(sockfd, (sockaddr *) &addr, sizeof(addr)) == -1) {
        fprintf(stderr, "connect failed, errno: %d, str: %s\n", errno, strerror(errno));
        close(sockfd);
        return -1;
    }
    printf("fd: %d, sock_name: %s, peer_name: %s\n", sockfd, get_sockname_string(sockfd).c_str(), get_peername_string(sockfd).c_str());

    vector<string> msgs = {"hello", "world", "cpp"};
    for (const auto &msg : msgs) {
        int n = send(sockfd, msg.c_str(), msg.size(), 0);
        if (n == -1) {
            fprintf(stderr, "send failed, errno: %d, str: %s\n", errno, strerror(errno));
            break;
        }
        printf("fd: %d, send %d bytes\n", sockfd, n);
    }
    close(sockfd);

    return 0;
}
```
### Read
#### 建连后客户端调用三次send，服务端在recv之前sleep 0秒
`server`:
```
$ ./server --sleep-before-recv 0
port: 27015, qlen: 0, sleep-before-recv: 0, sleep-after-listen: 0
getpeername failed, errno: 57, str: Socket is not connected
create socket fd: 3, sock_name: 0.0.0.0:0, peer_name:
getpeername failed, errno: 57, str: Socket is not connected
bind socket fd: 3, sock_name: 0.0.0.0:27015, peer_name:
getpeername failed, errno: 57, str: Socket is not connected
listen fd: 3, sock_name: 0.0.0.0:27015, peer_name:
getpeername failed, errno: 57, str: Socket is not connected
after accept sockfd: 3, sock_name: 0.0.0.0:27015, peer_name:
accepted fd: 4, sock_name: 127.0.0.1:27015, peer_name: 127.0.0.1:59932, addr: 127.0.0.1:59932
read 5 bytes from fd: 4, buf: hello
^C
```

`client`:
```
$ ./client
getpeername failed, errno: 57, str: Socket is not connected
fd: 3, sock_name: 0.0.0.0:0, peer_name:
fd: 3, sock_name: 127.0.0.1:59932, peer_name: 127.0.0.1:27015
fd: 3, send 5 bytes
fd: 3, send 5 bytes
fd: 3, send 3 bytes
```
#### 建连后客户端调用三次send，服务端在recv之前sleep 5秒
`server`:
```
$ ./server --sleep-before-recv 5
port: 27015, qlen: 0, sleep-before-recv: 5, sleep-after-listen: 0
getpeername failed, errno: 57, str: Socket is not connected
create socket fd: 3, sock_name: 0.0.0.0:0, peer_name:
getpeername failed, errno: 57, str: Socket is not connected
bind socket fd: 3, sock_name: 0.0.0.0:27015, peer_name:
getpeername failed, errno: 57, str: Socket is not connected
listen fd: 3, sock_name: 0.0.0.0:27015, peer_name:
getpeername failed, errno: 57, str: Socket is not connected
after accept sockfd: 3, sock_name: 0.0.0.0:27015, peer_name:
accepted fd: 4, sock_name: 127.0.0.1:27015, peer_name: 127.0.0.1:60018, addr: 127.0.0.1:60018
# 五秒后
read 13 bytes from fd: 4, buf: helloworldcpp
^C
```

`client`:
先执行服务端，再执行客户端，客服端执行完三次send后未等服务端read就已返回
```
$ ./client
getpeername failed, errno: 57, str: Socket is not connected
fd: 3, sock_name: 0.0.0.0:0, peer_name:
fd: 3, sock_name: 127.0.0.1:60018, peer_name: 127.0.0.1:27015
fd: 3, send 5 bytes
fd: 3, send 5 bytes
fd: 3, send 3 bytes
```

### Listen Backlog
#### backlog传2
`server`:
```
$ ./server --sleep-after-listen 10 --qlen 2
port: 27015, qlen: 2, sleep-before-recv: 0, sleep-after-listen: 10
getpeername failed, errno: 57, str: Socket is not connected
create socket fd: 3, sock_name: 0.0.0.0:0, peer_name:
getpeername failed, errno: 57, str: Socket is not connected
bind socket fd: 3, sock_name: 0.0.0.0:27015, peer_name:
getpeername failed, errno: 57, str: Socket is not connected
listen fd: 3, sock_name: 0.0.0.0:27015, peer_name:
# sleep here
getpeername failed, errno: 57, str: Socket is not connected
after accept sockfd: 3, sock_name: 0.0.0.0:27015, peer_name:
accepted fd: 4, sock_name: 127.0.0.1:27015, peer_name: 127.0.0.1:63689, addr: 127.0.0.1:63689
read 13 bytes from fd: 4, buf: helloworldcpp
getpeername failed, errno: 57, str: Socket is not connected
after accept sockfd: 3, sock_name: 0.0.0.0:27015, peer_name:
accepted fd: 4, sock_name: 127.0.0.1:27015, peer_name: 127.0.0.1:63690, addr: 127.0.0.1:63690
read 13 bytes from fd: 4, buf: helloworldcpp
getpeername failed, errno: 57, str: Socket is not connected
after accept sockfd: 3, sock_name: 0.0.0.0:27015, peer_name:
accepted fd: 4, sock_name: 127.0.0.1:27015, peer_name: 127.0.0.1:63691, addr: 127.0.0.1:63691
read 13 bytes from fd: 4, buf: helloworldcpp
^C
```

`client`:
```
$ ./client
getpeername failed, errno: 57, str: Socket is not connected
fd: 3, sock_name: 0.0.0.0:0, peer_name:
fd: 3, sock_name: 127.0.0.1:63689, peer_name: 127.0.0.1:27015
fd: 3, send 5 bytes
fd: 3, send 5 bytes
fd: 3, send 3 bytes
$ ./client
getpeername failed, errno: 57, str: Socket is not connected
fd: 3, sock_name: 0.0.0.0:0, peer_name:
fd: 3, sock_name: 127.0.0.1:63690, peer_name: 127.0.0.1:27015
fd: 3, send 5 bytes
fd: 3, send 5 bytes
fd: 3, send 3 bytes
$ ./client
getpeername failed, errno: 57, str: Socket is not connected
fd: 3, sock_name: 0.0.0.0:0, peer_name:
# block here
fd: 3, sock_name: 127.0.0.1:63691, peer_name: 127.0.0.1:27015
fd: 3, send 5 bytes
fd: 3, send 5 bytes
fd: 3, send 3 bytes
```

#### backlog传0
参数backlog提供了一个提示，提示系统该进程所要入队的未完成连接请求数量。其实际值由系统决定。具体的最大值取决于每个协议的实现。对于TCP，其默认值为128。[^backlog]
通过下面的脚本，在macOs上测试，验证了TCP的默认值为128。

`run-server.sh`
```bash
#!/bin/bash

for i in {1..256}; do
    echo "do $i"
    ./client 1>/dev/null 2>&1
    echo "$i finished"
done
```

`server`:
```
$ ./server --sleep-after-listen 10
port: 27015, qlen: 0, sleep-before-recv: 0, sleep-after-listen: 10
getpeername failed, errno: 57, str: Socket is not connected
create socket fd: 3, sock_name: 0.0.0.0:0, peer_name:
getpeername failed, errno: 57, str: Socket is not connected
bind socket fd: 3, sock_name: 0.0.0.0:27015, peer_name:
getpeername failed, errno: 57, str: Socket is not connected
listen fd: 3, sock_name: 0.0.0.0:27015, peer_name:
# sleep here
getpeername failed, errno: 57, str: Socket is not connected
after accept sockfd: 3, sock_name: 0.0.0.0:27015, peer_name:
accepted fd: 4, sock_name: 127.0.0.1:27015, peer_name: 127.0.0.1:63789, addr: 127.0.0.1:63789
read 13 bytes from fd: 4, buf: helloworldcpp
getpeername failed, errno: 57, str: Socket is not connected
after accept sockfd: 3, sock_name: 0.0.0.0:27015, peer_name:
accepted fd: 4, sock_name: 127.0.0.1:27015, peer_name: 127.0.0.1:63790, addr: 127.0.0.1:63790
read 13 bytes from fd: 4, buf: helloworldcpp
................
getpeername failed, errno: 57, str: Socket is not connected
after accept sockfd: 3, sock_name: 0.0.0.0:27015, peer_name:
accepted fd: 4, sock_name: 127.0.0.1:27015, peer_name: 127.0.0.1:64050, addr: 127.0.0.1:64050
read 5 bytes from fd: 4, buf: hello
getpeername failed, errno: 57, str: Socket is not connected
after accept sockfd: 3, sock_name: 0.0.0.0:27015, peer_name:
accepted fd: 4, sock_name: 127.0.0.1:27015, peer_name: 127.0.0.1:64051, addr: 127.0.0.1:64051
read 13 bytes from fd: 4, buf: helloworldcpp
^C
```

`client`:
```
$ ./run-server.sh
do 1
1 finished
do 2
2 finished
do 3
3 finished
do 4
4 finished
..................
do 126
126 finished
do 127
127 finished
do 128
128 finished
do 129
# block here
129 finished
do 130
130 finished
do 131
131 finished
do 132
132 finished
..................
do 255
255 finished
do 256
256 finished
```

### INADDR_ANY
对于因特网域，如果指定IP地址为INADDR_ANY(`<netinet/in.h>`中定义的)，套接字端点可以被绑定到所有的系统网络接口上。这意味着可以接收这个系统所安装的任何一个网卡的数据包。如果调用connect或listen，但没有将地址绑定到套接字上，系统会选择一个地址绑定到套接字上。[^getsockname]

可以注意到下面在`bind 0.0.0.0`后的sockfd在getsockname后得到的ip是`0.0.0.0`，在accept得到的fd上getsockname后得到的ip才是`127.0.0.1`或`other-ip`，而`bind other-ip`后的sockfd在getsockname后得到的ip就是`other-ip`。

#### Bind 0.0.0.0
```
$ ./server
$ ./server --ip 0.0.0.0
port: 27015, qlen: 0, sleep-before-recv: 0, sleep-after-listen: 0
getpeername failed, errno: 57, str: Socket is not connected
create socket fd: 3, sock_name: 0.0.0.0:0, peer_name:
getpeername failed, errno: 57, str: Socket is not connected
bind socket fd: 3, sock_name: 0.0.0.0:27015, peer_name:
getpeername failed, errno: 57, str: Socket is not connected
listen fd: 3, sock_name: 0.0.0.0:27015, peer_name:
getpeername failed, errno: 57, str: Socket is not connected
after accept sockfd: 3, sock_name: 0.0.0.0:27015, peer_name:
accepted fd: 4, sock_name: other-ip:27015, peer_name: other-ip:62771, addr: other-ip:62771
read 5 bytes from fd: 4, buf: hello
getpeername failed, errno: 57, str: Socket is not connected
after accept sockfd: 3, sock_name: 0.0.0.0:27015, peer_name:
accepted fd: 4, sock_name: 127.0.0.1:27015, peer_name: 127.0.0.1:62772, addr: 127.0.0.1:62772
read 13 bytes from fd: 4, buf: helloworldcpp
^C
```
```
$ ./client --ip other-ip
getpeername failed, errno: 57, str: Socket is not connected
fd: 3, sock_name: 0.0.0.0:0, peer_name:
fd: 3, sock_name: other-ip:62771, peer_name: other-ip:27015
fd: 3, send 5 bytes
fd: 3, send 5 bytes
$ ./client --ip 127.0.0.1
getpeername failed, errno: 57, str: Socket is not connected
fd: 3, sock_name: 0.0.0.0:0, peer_name:
fd: 3, sock_name: 127.0.0.1:62772, peer_name: 127.0.0.1:27015
fd: 3, send 5 bytes
fd: 3, send 5 bytes
fd: 3, send 3 bytes
```
#### Bind other-ip
```
$ ./server --ip other-ip
port: 27015, qlen: 0, sleep-before-recv: 0, sleep-after-listen: 0
getpeername failed, errno: 57, str: Socket is not connected
create socket fd: 3, sock_name: 0.0.0.0:0, peer_name:
getpeername failed, errno: 57, str: Socket is not connected
bind socket fd: 3, sock_name: other-ip:27015, peer_name:
getpeername failed, errno: 57, str: Socket is not connected
listen fd: 3, sock_name: other-ip:27015, peer_name:
getpeername failed, errno: 57, str: Socket is not connected
after accept sockfd: 3, sock_name: other-ip:27015, peer_name:
accepted fd: 4, sock_name: other-ip:27015, peer_name: other-ip:62773, addr: other-ip:62773
read 13 bytes from fd: 4, buf: helloworldcpp
^C
```
```
$ ./client --ip other-ip
getpeername failed, errno: 57, str: Socket is not connected
fd: 3, sock_name: 0.0.0.0:0, peer_name:
fd: 3, sock_name: other-ip:62773, peer_name: other-ip:27015
fd: 3, send 5 bytes
fd: 3, send 5 bytes
fd: 3, send 3 bytes
$ ./client --ip 127.0.0.1
getpeername failed, errno: 57, str: Socket is not connected
fd: 3, sock_name: 0.0.0.0:0, peer_name:
connect failed, errno: 61, str: Connection refused
```

[^getsockname]: 《UNIX环境高级编程》16.3.4
[^backlog]: 《UNIX环境高级编程》16.4