---
title: Redis笔记 info命令
date: 2019-12-11 16:22:00
tags: Redis
---

命令具体实现在`redis-3.0/src/redis.c:genRedisInfoString`。

# Memory
* `used_memory`是redis通过在每次执行`malloc`和`free`等函数的时候维护定义在`src/zmalloc.c`中的`used_memory`变量实现的
* `used_memory_rss`在linux中是通过读`/proc/{pid}/stat`这个文件的第24个字段`rss`得到number of pages the process  in real memory然后再乘以`sysconf(_SC_PAGESIZE)`实现的。`sysconf(_SC_PAGESIZE)`表示Size of a page in bytes。
* `used_memory_peak`Record the max memory used since the server was started.
* `mem_fragmentation_ratio`Fragmentation = RSS / allocated-bytes，allocated-bytes即为`used_memory`

# CPU
* `used_cpu_user`调用`getrusage`得到的`ru_utime`
* `used_cpu_sys`调用`getrusage`得到的`ru_stime`