---
title: Heap Sort
date: 2020-11-01 22:06:56
tags:
- Algorithm
---

```cpp
#include <bits/stdc++.h>
#define DBG(x) cerr << #x << " = " << x << endl

using namespace std;
typedef long long LL;

void heap_sort(vector<int> &vec) {
    for (int i = 0; i < vec.size(); i++) {
        for (int p = i; p && vec[(p - 1) / 2] < vec[p]; p = (p - 1) / 2) {
            swap(vec[p], vec[(p - 1) / 2]);
        }
    }

    for (int i = vec.size() - 1; i >= 0; i--) {
        int top = vec[0];
        vec[0] = vec[i];

        for (int p = 0, c; (c = p * 2 + 1) < i; p = c) {
            if (c + 1 < i && vec[c] < vec[c + 1]) c++;
            if (vec[c] < vec[p]) break;
            swap(vec[c], vec[p]);
        }

        vec[i] = top;
    }
}

int main(int argc, char **argv) {
    default_random_engine rd(time(NULL));

    for (int cas = 0; cas < 100; cas++) {
        vector<int> vec;
        for (int i = 0; i < 100000; i++) {
            vec.push_back(rd());
        }
        auto stl = vec;
        heap_sort(vec);
        sort(stl.begin(), stl.end());
        assert(vec == stl);
    }

    return 0;
}
```