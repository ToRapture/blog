---
title: UNIX编程 文件I/O笔记
date: 2019-11-28 11:25:00
mathjax: true
tags:
- UNIX编程
---

## 打开、创建、关闭文件
`test_create.cpp`:
```cpp
#include <cstdio>

#include <unistd.h>
#include <fcntl.h>

using namespace std;

int main(int argc, char **argv) {
    // 读、写打开，若不存在则创建，若已存在则出错
    int flags = O_RDWR | O_CREAT | O_EXCL;
    // 0644
    mode_t mode = S_IRUSR | S_IWUSR | S_IRGRP | S_IROTH;

    int fd = open("test_create.txt", flags, mode);
    printf("fd: %d\n", fd);
    if (fd != -1) {
        close(fd);
    }
    return 0;
}
```
```
$ ls test_create.txt
ls: test_create.txt: No such file or directory
$ ./test_create
fd: 3
$ ls -lh test_create.txt
-rw-r--r--  1 torapture  staff     0B 11 27 20:01 test_create.txt
$ ./test_create
fd: -1
```
由open函数返回的文件描述符一定是最小的未用描述符取值。[^min_fd]

## 读写
`rw.cpp`:
```cpp
#include <cstdio>

#include <vector>
#include <string>

#include <unistd.h>
#include <fcntl.h>

using namespace std;

int main(int argc, char **argv) {
    int flags = O_RDWR | O_CREAT;
    mode_t mode = S_IRUSR | S_IWUSR | S_IRGRP | S_IROTH;

    int fd = open("rw.txt", flags, mode);
    if (fd == -1) {
        printf("open file error\n");
        return 1;
    }

    printf("file opened, fd[%d]\n", fd);
    printf("cur_pos[%d]\n", lseek(fd, 0, SEEK_CUR));
    vector<string> msgs = {"hello", "world", "cpp"};
    for (const auto &msg : msgs) {
        int n = write(fd, msg.c_str(), msg.size());
        int cur_pos = lseek(fd, 0, SEEK_CUR);
        printf("write msg[%s] to fd[%d], n[%d], cur_pos[%d]\n", msg.c_str(), fd, n, cur_pos);
    }
    close(fd);
    return 0;
}
```
```
$ ./rw
cur_pos[0]
file opened, fd[3]
write msg[hello] to fd[3], n[5], cur_pos[5]
write msg[world] to fd[3], n[5], cur_pos[10]
write msg[cpp] to fd[3], n[3], cur_pos[13]
```

## 偏移量与文件共享
在Linux系统中，若两个进程A与B同时打开某个文件F，在`/proc/[pid]/fdinfo/[fd]`[^proc]中可以看到各自的文件偏移量。
`dup.cpp`:
```cpp
#include <cstdio>

#include <vector>
#include <string>

#include <unistd.h>
#include <fcntl.h>

using namespace std;

int main(int argc, char **argv) {
    int flags = O_RDWR | O_CREAT;
    mode_t mode = S_IRUSR | S_IWUSR | S_IRGRP | S_IROTH;

    int fd = open("dup.txt", flags, mode);
    if (fd == -1) {
        printf("open file error\n");
        return 1;
    }

    printf("file opened, fd[%d]\n", fd);

    int f2 = dup(fd);
    if (f2 == -1) {
        printf("dup fd[%d] failed\n", fd);
        close(fd);
        return 1;
    }

    printf("cur_pos of %d is %d\n", fd, lseek(fd, 0, SEEK_CUR));
    printf("cur_pos of %d is %d\n", f2, lseek(f2, 0, SEEK_CUR));

    vector<string> msgs = {"hello", "world", "cpp"};
    for (const auto &msg : msgs) {
        int n;
        n = write(fd, msg.c_str(), msg.size());
        printf("write msg[%s] to fd[%d], n[%d]\n", msg.c_str(), fd, n);
        printf("cur_pos of %d is %d\n", fd, lseek(fd, 0, SEEK_CUR));
        printf("cur_pos of %d is %d\n", f2, lseek(f2, 0, SEEK_CUR));

        n = write(f2, msg.c_str(), msg.size());
        printf("write msg[%s] to fd[%d], n[%d]\n", msg.c_str(), f2, n);
        printf("cur_pos of %d is %d\n", fd, lseek(fd, 0, SEEK_CUR));
        printf("cur_pos of %d is %d\n", f2, lseek(f2, 0, SEEK_CUR));
    }

    close(f2);
    close(fd);
    return 0;
}
```
```
$ ./dup
file opened, fd[3]
cur_pos of 3 is 0
cur_pos of 4 is 0
write msg[hello] to fd[3], n[5]
cur_pos of 3 is 5
cur_pos of 4 is 5
write msg[hello] to fd[4], n[5]
cur_pos of 3 is 10
cur_pos of 4 is 10
write msg[world] to fd[3], n[5]
cur_pos of 3 is 15
cur_pos of 4 is 15
write msg[world] to fd[4], n[5]
cur_pos of 3 is 20
cur_pos of 4 is 20
write msg[cpp] to fd[3], n[3]
cur_pos of 3 is 23
cur_pos of 4 is 23
write msg[cpp] to fd[4], n[3]
cur_pos of 3 is 26
cur_pos of 4 is 26
```
dup函数返回的新文件描述符与参数`fd`共享同一个文件表项[^dup_fd]，对文件描述符`fd`和`f2`分别写入后`fd`和`f2`的当前文件偏移量相等。

[^min_fd]: 《UNIX环境高级编程》3.3
[^proc]: proc - process information pseudo-filesystem http://man7.org/linux/man-pages/man5/proc.5.html
[^dup_fd]: 《UNIX环境高级编程》3.12