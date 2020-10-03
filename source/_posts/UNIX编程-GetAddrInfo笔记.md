---
title: UNIX编程 GetAddrInfo笔记
date: 2019-12-06 15:58:00
tags: UNIX编程
---

实现参考：
* 《UNIX环境高级编程》16.3.3 地址查询
* 《UNIX系统编程手册 下》59.10.1
* man getaddrinfo

`addr`:
```cpp
#include <cstdio>
#include <cstring>

#include <iostream>
#include <string>

#include <sys/socket.h>
#include <netdb.h>

using namespace std;

string getnameinfo_str(const sockaddr *addr, socklen_t addrlen, int flags) {
    char host[NI_MAXHOST];
    char service[NI_MAXSERV];
    int ret = getnameinfo(addr, addrlen, host, sizeof(host), service, sizeof(service), flags);
    if (ret != 0) {
        fprintf(stderr, "getnameinfo failed, ret: %d, str: %s\n", ret, gai_strerror(ret));
    }
    return host + string(":") + service;
}

void work(const string &host, const string &service, int hints_flags, int getaddrinfo_flags) {
    printf("host: %s, service: %s, hints_flags: %d, getaddrinfo_flags: %d\n", host.c_str(), service.c_str(), hints_flags, getaddrinfo_flags);
    addrinfo hints;
    addrinfo *result;
    memset(&hints, 0, sizeof(hints));

    hints.ai_socktype = SOCK_STREAM;
    hints.ai_family = AF_UNSPEC;
    hints.ai_flags = hints_flags;

    int ret = getaddrinfo(host == "-" ? NULL : host.c_str(), service == "-" ? NULL : service.c_str(), &hints, &result);
    if (ret != 0) {
        fprintf(stderr, "getaddrinfo, host: %s, service: %s, ret: %d, str: %s\n", host.c_str(), service.c_str(), ret, gai_strerror(ret));
        return;
    }
    for (addrinfo *p = result; p != NULL; p = p->ai_next) {
        printf("addr: %s, flags: %d, family: %d, socktype: %d, procotol: %d\n",
               getnameinfo_str(p->ai_addr, p->ai_addrlen, getaddrinfo_flags).c_str(), p->ai_flags, p->ai_family, p->ai_socktype, p->ai_protocol);
        if (p == result && (hints_flags & AI_CANONNAME)) {
            printf("canonname: %s\n", p->ai_canonname);
        }
    }

    freeaddrinfo(result);

    puts("-----------");
}

int main(int argc, char **argv) {
    string host, service;
    int hints_flags, getaddrinfo_flags;
    while (cin >> host >> service >> hints_flags >> getaddrinfo_flags) {
        work(host, service, hints_flags, getaddrinfo_flags);
    }

    work("-", "https", AI_PASSIVE, 0);
    work("-", "https", 0, 0);
    work("-", "https", AI_PASSIVE | AI_CANONNAME, 0);

    work("127.0.0.1", "https", AI_PASSIVE, 0);
    work("127.0.0.1", "https", AI_PASSIVE | AI_CANONNAME, 0);
    work("127.0.0.1", "https", AI_PASSIVE | AI_CANONNAME, NI_NUMERICHOST | NI_NUMERICSERV);

    work("localhost", "http", AI_CANONNAME, 0);
    work("localhost", "http", AI_CANONNAME, NI_NUMERICHOST | NI_NUMERICSERV);

    work("baidu.com", "http", AI_CANONNAME, 0);
    work("baidu.com", "http", AI_CANONNAME, NI_NUMERICHOST | NI_NUMERICSERV);

    work("google.com", "http", AI_CANONNAME, 0);
    work("google.com", "http", AI_CANONNAME, NI_NUMERICHOST | NI_NUMERICSERV);

    return 0;
}
```

```
$ ./addr
host: -, service: https, hints_flags: 1, getaddrinfo_flags: 0
addr: :::https, flags: 0, family: 30, socktype: 1, procotol: 6
addr: 0.0.0.0:https, flags: 0, family: 2, socktype: 1, procotol: 6
-----------
host: -, service: https, hints_flags: 0, getaddrinfo_flags: 0
addr: localhost:https, flags: 0, family: 30, socktype: 1, procotol: 6
addr: localhost:https, flags: 0, family: 2, socktype: 1, procotol: 6
-----------
host: -, service: https, hints_flags: 3, getaddrinfo_flags: 0
addr: :::https, flags: 0, family: 30, socktype: 1, procotol: 6
canonname: localhost
addr: 0.0.0.0:https, flags: 0, family: 2, socktype: 1, procotol: 6
-----------
host: 127.0.0.1, service: https, hints_flags: 1, getaddrinfo_flags: 0
addr: localhost:https, flags: 0, family: 2, socktype: 1, procotol: 6
-----------
host: 127.0.0.1, service: https, hints_flags: 3, getaddrinfo_flags: 0
addr: localhost:https, flags: 0, family: 2, socktype: 1, procotol: 6
canonname: (null)
-----------
host: 127.0.0.1, service: https, hints_flags: 3, getaddrinfo_flags: 10
addr: 127.0.0.1:443, flags: 0, family: 2, socktype: 1, procotol: 6
canonname: (null)
-----------
host: localhost, service: http, hints_flags: 2, getaddrinfo_flags: 0
addr: localhost:http, flags: 0, family: 30, socktype: 1, procotol: 6
canonname: localhost
addr: localhost:http, flags: 0, family: 2, socktype: 1, procotol: 6
-----------
host: localhost, service: http, hints_flags: 2, getaddrinfo_flags: 10
addr: ::1:80, flags: 0, family: 30, socktype: 1, procotol: 6
canonname: localhost
addr: 127.0.0.1:80, flags: 0, family: 2, socktype: 1, procotol: 6
-----------
host: baidu.com, service: http, hints_flags: 2, getaddrinfo_flags: 0
addr: 220.181.38.148:http, flags: 0, family: 2, socktype: 1, procotol: 6
canonname: baidu.com
addr: 39.156.69.79:http, flags: 0, family: 2, socktype: 1, procotol: 6
-----------
host: baidu.com, service: http, hints_flags: 2, getaddrinfo_flags: 10
addr: 220.181.38.148:80, flags: 0, family: 2, socktype: 1, procotol: 6
canonname: baidu.com
addr: 39.156.69.79:80, flags: 0, family: 2, socktype: 1, procotol: 6
-----------
host: google.com, service: http, hints_flags: 2, getaddrinfo_flags: 0
addr: hkg07s22-in-f110.1e100.net:http, flags: 0, family: 2, socktype: 1, procotol: 6
canonname: google.com
addr: hkg07s22-in-x0e.1e100.net:http, flags: 0, family: 30, socktype: 1, procotol: 6
-----------
host: google.com, service: http, hints_flags: 2, getaddrinfo_flags: 10
addr: 216.58.199.110:80, flags: 0, family: 2, socktype: 1, procotol: 6
canonname: google.com
addr: 2404:6800:4005:803::200e:80, flags: 0, family: 30, socktype: 1, procotol: 6
-----------
```