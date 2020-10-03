---
title: UDP headers and checksum
date: 2020-04-27 14:03:00
mathjax: true
tags:
- Network
- TCP/IP
---

## UDP headers
![](https://img2020.cnblogs.com/blog/1224734/202004/1224734-20200427140017732-1635616137.png)

## Headers for computing checksum
![](https://img2020.cnblogs.com/blog/1224734/202004/1224734-20200427140024136-683956251.png)

The checksum computation is similar to the Internet checksum computation.

![](https://img2020.cnblogs.com/blog/1224734/202004/1224734-20200427140101858-948940853.png)

```cpp
#include <bits/stdc++.h>
#define DBG(x) cerr << #x << " = " << x << endl

using namespace std;
typedef long long LL;

unsigned short checksum(const vector<unsigned short> &v) {
    unsigned int sum = 0;
    for (int i = 0; i < v.size(); i++) {
        sum += v[i];
    }
    while (sum > 0xFFFF) {
        sum = (sum & 0xFFFF) + (sum >> 16);
    }
    return ~sum;
}

int main(int argc, char **argv) {
    vector<unsigned short> vec{
            // UDP pseudo-header
            0x0a01, 0xbf44, // source IP address
            0x0a02, 0x0002, // destination IP address
            0x0011 /* 1 byte zero and 1 byte protocol number */, 0x0023, // UDP length

            // UDP header
            0xc183, 0x0035, // source port number and destination port number
            0x0000, 0x0023, // checksum and UDP length

            // data
            0x06b0, 0x0100,
            0x0001, 0x0000,
            0x0000, 0x0000,
            0x0562, 0x6169,
            0x6475, 0x0363,
            0x6f6d, 0x0000,
            0x0100, 0x0100, // the length of this UDP packet is odd and the last short is 0x01, so we should pad 1 byte zero.

    };
    unsigned short sum = checksum(vec);

    printf("checksum = 0x%X\n", sum);
    assert(sum == 0x22E4);

    vec[8] = sum;

    sum = checksum(vec);
    assert(sum == 0);
    printf("0x%X\n", sum);

    return 0;
}
```