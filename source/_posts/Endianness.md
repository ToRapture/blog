---
title: Endianness
date: 2020-05-09 16:46:00
tags:
- Network
---

# What is endianness?
Little and big endian are two ways of storing multibyte data-types ( int, float, etc). In little endian machines, last byte of binary representation of the multibyte data-type is stored first. On the other hand, in big endian machines, first byte of binary representation of the multibyte data-type is stored first.

# How to check
```cpp
#include <bits/stdc++.h>
#define DBG(x) cerr << #x << " = " << x << endl

using namespace std;
typedef long long LL;


string get_endian() {
    union {
        short s;
        char c[sizeof(short)];
    } un;
    un.s = 0x1020;

    if (sizeof(short) != 2)
        return "sizeof(short) != 2";

    if (un.c[0] == 0x20 && un.c[1] == 0x10)
        return "little-endian";
    if (un.c[0] == 0x10 && un.c[1] == 0x20)
        return "big-endian";
    return "unknown";

}

int main(int argc, char **argv) {
    cout << "endian: " << get_endian() << "\n";
    return 0;
}

```

# Network byte order
https://stackoverflow.com/a/997586/13133551
>"Network byte order" is Big Endian, and protocols such as TCP use this for integer fields (e.g. port numbers). Functions such as htons and ntohs can be used to do conversion.

>The data itself doesn't have any endianness it's entirely application defined.

![](https://img2020.cnblogs.com/blog/1224734/202005/1224734-20200509164613286-1013916493.png)
As we can see from the picture, the endianness of net is Big Endian.