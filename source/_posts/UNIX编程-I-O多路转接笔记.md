---
title: UNIX编程 I/O多路转接笔记
date: 2019-11-29 14:59:00
tags: UNIX编程
---

## 在标准输入上测试select
实现参考了《UNIX环境高级编程》14.4.1和《UNIX系统编程手册》63.2.1。
```cpp
#include <cstdio>
#include <iostream>
#include <unistd.h>
#include <fcntl.h>
#include <sys/select.h>

using namespace std;

const int BUF_SIZE = 128;

int main(int argc, char **argv) {
    fd_set rs;
    FD_ZERO(&rs);
    FD_SET(STDIN_FILENO, &rs);

    for (int i = 0; i < 5; i++) {
        timeval tv = {.tv_sec=2, .tv_usec=0};
        fd_set rs_temp = rs;
        int ret = select(STDIN_FILENO + 1, &rs_temp, NULL, NULL, &tv);
        if (ret == -1) {
            printf("select ret: %d, errno: %d\n", ret, errno);
            return 1;
        }
        printf("select ret: %d\n", ret);
        printf("if can read data from fd: %d is %s\n", STDIN_FILENO, FD_ISSET(STDIN_FILENO, &rs_temp) ? "true" : "false");
        if (FD_ISSET(STDIN_FILENO, &rs_temp)) {
            char buf[BUF_SIZE];
            int n = read(STDIN_FILENO, buf, 5);
            if (n == -1) {
                printf("read failed, errno: %d\n", errno);
                return 1;
            }
            buf[n] = '\0';
            printf("success read %d bytes, buf: %s\n", n, buf);
        }
    }

    return 0;
}
```
下面运行结果中第三行的`hello world`是在第一次循环结果打印后，第二次调用select阻塞时输入的。
运行结果：
```
select ret: 0
if can read data from fd: 0 is false
hello world
select ret: 1
if can read data from fd: 0 is true
success read 5 bytes, buf: hello
select ret: 1
if can read data from fd: 0 is true
success read 5 bytes, buf:  worl
select ret: 1
if can read data from fd: 0 is true
success read 2 bytes, buf: d

select ret: 0
if can read data from fd: 0 is false

Process finished with exit code 0
```


## 非阻塞I/O
一个描述符阻塞与否并不影响select是否阻塞。[^select_block_fd]
```cpp
#include <cstdio>
#include <cstring>
#include <iostream>
#include <unistd.h>
#include <fcntl.h>
#include <errno.h>

const int BUF_SIZE = 128;

using namespace std;

void set_fl(int fd, int flags) {
    int val;
    if ((val = fcntl(fd, F_GETFL, 0)) < 0) {
        fprintf(stderr, "fcntl F_GETFL error\n");
        exit(1);
    }
    val |= flags;
    if (fcntl(fd, F_SETFL, val) < 0) {
        fprintf(stderr, "fcntl F_SETFL error\n");
        exit(1);
    }
}

int main(int argc, char **argv) {
    int n;
    char buf[128];

    set_fl(STDIN_FILENO, O_NONBLOCK);
    n = read(STDIN_FILENO, buf, BUF_SIZE - 1);
    printf("n: %d, errno: %d, str: %s\n", n, errno, strerror(errno));

    fd_set rs;
    FD_ZERO(&rs);
    FD_SET(STDIN_FILENO, &rs);

    for (int i = 0; i < 5; i++) {
        fd_set rs_temp = rs;
        timeval tv = {.tv_sec=2, .tv_usec=0};
        int ret = select(STDIN_FILENO + 1, &rs_temp, NULL, NULL, &tv);
        if (ret == -1) {
            fprintf(stderr, "select ret: %d, errno: %d, str: %s\n", ret, errno, strerror(errno));
            return 1;
        }
        printf("select ret: %d\n", ret);
        printf("if can read data from fd: %d is %s\n", STDIN_FILENO, FD_ISSET(STDIN_FILENO, &rs_temp) ? "true" : "false");
    }

    return 0;
}
```
执行结果：
```
n: -1, errno: 35, str: Resource temporarily unavailable
select ret: 0
if can read data from fd: 0 is false
select ret: 0
if can read data from fd: 0 is false
select ret: 0
if can read data from fd: 0 is false
select ret: 0
if can read data from fd: 0 is false
select ret: 0
if can read data from fd: 0 is false

Process finished with exit code 0
```
可以看到代码首先对设置了非阻塞标志的标准输入读，read函数返回-1，每次select的返回结果也是0。


## select阻塞中被信号中断
```cpp
#include <cstdio>
#include <cstring>
#include <iostream>
#include <unistd.h>
#include <fcntl.h>
#include <errno.h>
#include <sys/select.h>

using namespace std;

void handler(int sig) {
    printf("caught signal: %d, str: %s\n", sig, strsignal(sig));
}

int main(int argc, char **argv) {
    signal(SIGUSR1, handler);

    int ret = select(0, NULL, NULL, NULL, NULL);
    printf("select ret: %d\n", ret);
    if (ret == -1) {
        printf("errno: %d, str: %s\n", errno, strerror(errno));
    }

    return 0;
}
```
运行后给进程发`SIGUSR1`信号。
```
caught signal: 30, str: User defined signal 1: 30
select ret: -1
errno: 4, str: Interrupted system call
```


## select支持的最大数量
```cpp
#include <cstdio>
#include <cstring>
#include <iostream>
#include <unistd.h>
#include <fcntl.h>
#include <errno.h>
#include <sys/select.h>

using namespace std;

int main(int argc, char **argv) {
    timeval tv;
    int ret;

    printf("FD_SETSIZE: %d\n", FD_SETSIZE);

    tv = {.tv_sec=0, .tv_usec=0};
    ret = select(FD_SETSIZE, NULL, NULL, NULL, &tv);
    printf("select ret: %d\n", ret);
    if (ret == -1) {
        printf("errno: %d, str: %s\n", errno, strerror(errno));
    }

    tv = {.tv_sec=0, .tv_usec=0};
    ret = select(FD_SETSIZE + 1, NULL, NULL, NULL, &tv);
    printf("select ret: %d\n", ret);
    if (ret == -1) {
        printf("errno: %d, str: %s\n", errno, strerror(errno));
    }

    return 0;
}
```
```
FD_SETSIZE: 1024
select ret: 0
select ret: -1
errno: 22, str: Invalid argument

Process finished with exit code 0
```
在本机的`macOs`上`$ uname -a`的结果为`Darwin localhost 19.0.0 Darwin Kernel Version 19.0.0: Thu Oct 17 16:17:15 PDT 2019; root:xnu-6153.41.3~29/RELEASE_X86_64 x86_64`
> The behavior of these macros is undefined if a descriptor value is less than zero or greater than or equal to FD\_SETSIZE, which is normally at least equal to the maximum number of descriptors supported by the system.[^man_select]


[^select_block_fd]: 《UNIX环境高级编程》14.4.1
[^man_select]: SELECT(2) BSD System Calls Manual DESCRIPTION