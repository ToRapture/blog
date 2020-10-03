---
title: 获取CPU时间和计算CPU使用率
date: 2019-12-15 14:48:00
tags:
- UNIX编程
---

```cpp
#include <bits/stdc++.h>
#include <unistd.h>
#include <sys/resource.h>
#include <sys/time.h>

#define DBG(x) cerr << #x << " = " << x << endl

using namespace std;
typedef long long LL;

double cpu_load(double start, double end, double used) {
    return used / (end - start) * 100;
}

double cpu_time_used_s() {
    rusage usage;
    getrusage(RUSAGE_SELF, &usage);
    return usage.ru_stime.tv_sec + double(usage.ru_stime.tv_usec) / 1000000 + usage.ru_utime.tv_sec + double(usage.ru_utime.tv_usec) / 1000000;
}

double get_time_s() {
    timeval tv;
    gettimeofday(&tv, NULL);
    return tv.tv_sec + double(tv.tv_usec) / 1000000;
}

int main(int argc, char **argv) {
    double start = get_time_s();

    srand(time(NULL));
    int x = rand();
    for (int i = 0; i < 300000000; i++) {
        x ^= rand();
        x |= rand();
        x &= ~rand();
    }

    sleep(5);

    double end = get_time_s();
    double real_time_used = end - start;
    double cpu_time_used = cpu_time_used_s();

    printf("start: %.3fs, end: %.3fs\n"
           "real_time_used: %.3f\n"
           "cpu_time_used: %.3fs, cpu_load: %.3f%%\n",
           start, end, real_time_used, cpu_time_used, cpu_load(start, end, cpu_time_used));

    return 0;
}
```

```
start: 1576392425.566s, end: 1576392438.229s
real_time_used: 12.663
cpu_time_used: 6.230s, cpu_load: 49.201%
```