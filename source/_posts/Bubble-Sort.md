---
title: Bubble Sort
date: 2020-11-01 21:53:23
tags:
- Algorithm
---

```cpp
#include <bits/stdc++.h>
#define DBG(x) cerr << #x << " = " << x << endl

using namespace std;
typedef long long LL;

void bubble_sort(vector<int> &vec) {
    for (int i = 0; i < vec.size() - 1; i++) {
        for (int j = vec.size() - 1; j > i; j--) {
            if (vec[j] < vec[j - 1]) {
                swap(vec[j], vec[j - 1]);
            }
        }
    }
}

int main(int argc, char **argv) {
    default_random_engine rd(time(NULL));

    for (int cas = 0; cas < 100; cas++) {
        vector<int> vec;
        for (int i = 0; i < 1000; i++) {
            vec.push_back(rd());
        }
        auto stl = vec;
        bubble_sort(vec);
        sort(stl.begin(), stl.end());
        assert(vec == stl);
    }

    return 0;
}
```