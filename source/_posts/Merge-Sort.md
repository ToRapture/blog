---
title: Merge Sort
date: 2020-11-01 21:53:42
tags:
- Algorithm
---

```cpp
#include <bits/stdc++.h>
#define DBG(x) cerr << #x << " = " << x << endl

using namespace std;
typedef long long LL;

void ms(int l, int r, vector<int> &vec, vector<int> &temp) {
    if (l >= r) return;
    int mid = l + (r - l) / 2;
    ms(l, mid, vec, temp);
    ms(mid + 1, r, vec, temp);
    int cnt = 0;

    for (int i = l, j = mid + 1; i <= mid || j <= r;) {
        if (i > mid) {
            temp[cnt++] = vec[j++];
        } else if (j > r) {
            temp[cnt++] = vec[i++];
        } else if (vec[i] <= vec[j]) {
            temp[cnt++] = vec[i++];
        } else {
            temp[cnt++] = vec[j++];
        }
    }

    for (int i = 0, j = l; i < cnt; i++, j++) {
        vec[j] = temp[i];
    }
}

void merge_sort(vector<int> &vec) {
    vector<int> temp(vec.size());
    ms(0, vec.size() - 1, vec, temp);
}

int main(int argc, char **argv) {
    default_random_engine rd(time(NULL));

    for (int cas = 0; cas < 100; cas++) {
        vector<int> vec;
        for (int i = 0; i < 100000; i++) {
            vec.push_back(rd());
        }
        auto stl = vec;
        merge_sort(vec);
        sort(stl.begin(), stl.end());
        assert(vec == stl);
    }
    return 0;
}
```