---
title: IP checksum
date: 2020-04-23 14:22:00
tags: Network, TCP/IP
---

To do an experiment, first capture an IP packet, as we can see the checksum is 0xCAD7. Then we set the checksum field to 0, and calculate the checksum, then we will get 0xCAD7. Set 0xCAD7 to the checksum field and calculate the checksum again, we will get 0 then.
![](https://img2020.cnblogs.com/blog/1224734/202004/1224734-20200423142202721-1109206249.png)

```cpp
#include <bits/stdc++.h>
#define DBG(x) cerr << #x << " = " << x << endl

using namespace std;
typedef long long LL;

unsigned short ip_checksum(const vector<unsigned short> &v) {
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

    vector<unsigned short> vec{0x4500, 0x0235, 0x0000, 0x4000, 0x4006, 0x0000, 0x0a01, 0xbf44, 0x8ba2, 0x1904};
    unsigned short sum = ip_checksum(vec);
    assert(sum == 0xcad7);
    printf("checksum = 0x%X\n", sum);

    vec[5] = sum;

    sum = ip_checksum(vec);
    assert(sum == 0);
    printf("%X\n", sum);
    return 0;
}
```