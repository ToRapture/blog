---
title: Manacher算法
date: 2019-11-14 14:57:00
mathjax: true
tags:
- Algorithm
---

[[kuangbin专题] Manacher](https://vjudge.net/contest/284138)

实现参考了<https://oi-wiki.org/string/manacher/>。

[POJ 3974 Palindrome](http://poj.org/problem?id=3974)
求字符串$A$的最长回文子串模板题，首先在$A$的第一个字符的左面和$A$的每个字符的后面填充不在输入范围内的字符`$`，得到串$S$，然后处理串$S$。
原串$A$长度为$n$，处理后的串$S$长度为$m=2\times n - 1$。
对于每个$i \in [0, m)$，试图计算在串$S$中以$i$为中心的最长回文子串的最大半径$p_i$，其中$p_i \ge 1$。
在对于每一个$i-1$计算$p_{i-1}$结束后，维护出一个$[l,r]$二元组，表示$i-1$位置处理结束后，$i$位置处理之前当前得到的回文子串中右边界$r$最大的回文子串，并且这个回文子串$[l, r]$是以$r$为右边界的最长的回文子串。

```cpp
#include <cstring>
#include <cstdio>
#include <algorithm>
#include <string>

using namespace std;
typedef long long LL;

const int MAX_N = 1000000 + 16;

struct Manacher {
    static const int $ = -1;
    int len_b;
    int b[2 * MAX_N + 1];
    int p[2 * MAX_N + 1];

    void init(char a[], int n) {
        len_b = 2 * n + 1;
        for (int i = 0; i < n; i++) {
            b[2 * i] = $;
            b[2 * i + 1] = a[i];
        }
        b[len_b - 1] = $;

        int l = 0;
        int r = -1;
        for (int i = 0; i < len_b; i++) {
            if (i > r) {
                p[i] = 1;
            } else {
                p[i] = min(p[r - i + l], r - i + 1);
            }
            while (i + p[i] < len_b && i - p[i] >= 0 && b[i + p[i]] == b[i - p[i]]) {
                p[i]++;
            }
            if (i + p[i] - 1 > r) {
                l = i - p[i] + 1;
                r = i + p[i] - 1;
            }
        }
    }

    int max_length() {
        int ans = 1;
        for (int i = 0; i < len_b; i++) {
            int temp = p[i];
            if (b[i] == $ && p[i] % 2 == 1) temp--;
            if (b[i] != $ && p[i] % 2 == 0) temp--;
            ans = max(ans, temp);
        }
        return ans;
    }
} manacher;

char s[MAX_N];

int main(int argc, char **argv) {
    int cas = 1;
    while (true) {
        scanf("%s", s);
        if (s == string("END")) break;
        int len = strlen(s);
        manacher.init(s, len);
        printf("Case %d: %d\n", cas++, manacher.max_length());
    }
    return 0;
}

/**
abcbabcbabcba
abacacbaaaab
END

Case 1: 13
Case 2: 6
*/
```

[HDU 4513](http://acm.hdu.edu.cn/showproblem.php?pid=4513)
最长不递减回文子串，注意扩展长度的时候比较一下大小。
```cpp
#include <cstring>
#include <cstdio>
#include <algorithm>
#include <string>

using namespace std;
typedef long long LL;

const int MAX_N = 1000000 + 16;

struct Manacher {
    static const int $ = -1;
    int b[2 * MAX_N + 1];
    int len_b;
    int p[2 * MAX_N + 1];

    void init(int a[], int n) {
        len_b = 2 * n + 1;
        for (int i = 0; i < n; i++) {
            b[2 * i] = $;
            b[2 * i + 1] = a[i];
        }
        b[len_b - 1] = $;

        int l = 0;
        int r = -1;
        for (int i = 0; i < len_b; i++) {
            if (i > r) {
                p[i] = 1;
            } else {
                p[i] = min(p[l + r - i], r - i + 1);
            }
            while (0 <= i - p[i] && i + p[i] < len_b && b[i - p[i]] == b[i + p[i]]) {
                if (b[i - p[i]] != $ && i - p[i] + 2 <= i && b[i - p[i]] > b[i - p[i] + 2]) break;
                p[i]++;
            }
            if (i + p[i] - 1 > r) {
                l = i - p[i] + 1;
                r = i + p[i] - 1;
            }
        }
    }

    int max_length() {
        int ans = 0;
        for (int i = 0; i < len_b; i++)
            ans = max(ans, p[i]);
        return ans - 1;
    }
} manacher;

int a[MAX_N];

int main(int argc, char **argv) {
    int T;
    scanf("%d", &T);
    while (T--) {
        int n;
        scanf("%d", &n);
        for (int i = 0; i < n; i++)
            scanf("%d", &a[i]);
        manacher.init(a, n);
        printf("%d\n", manacher.max_length());
    }
    return 0;
}

/**
100
3
51 52 51
4
51 52 52 51
10
1 2 3 3 3 4 4 4 4 4

3
4
*/
```

[HDU 3294](http://acm.hdu.edu.cn/showproblem.php?pid=3294)
先把原串$A$的每个字符再变换一下，然后输出最长回文子串。
先在$S$中找到最长回文子串的长度和左端点，然后再通过$A$来输出。
```cpp
#include <cstring>
#include <cstdio>
#include <algorithm>
#include <string>

using namespace std;
typedef long long LL;

const int MAX_N = 1000000 + 16;

struct Manacher {
    static const int $ = -1;
    int b[2 * MAX_N + 1];
    int len_b;
    int p[2 * MAX_N + 1];

    void init(char a[], int n) {
        len_b = 2 * n + 1;
        for (int i = 0; i < n; i++) {
            b[2 * i] = $;
            b[2 * i + 1] = a[i];
        }
        b[len_b - 1] = $;

        int l = 0;
        int r = -1;
        for (int i = 0; i < len_b; i++) {
            if (i > r) {
                p[i] = 1;
            } else {
                p[i] = min(p[l + r - i], r - i + 1);
            }
            while (0 <= i - p[i] && i + p[i] < len_b && b[i - p[i]] == b[i + p[i]]) {
                p[i]++;
            }
            if (i + p[i] - 1 > r) {
                l = i - p[i] + 1;
                r = i + p[i] - 1;
            }
        }
    }

    pair<int, int> max_length() {
        int ans = 0;
        int l = -1;
        for (int i = 0; i < len_b; i++) {
            if (p[i] - 1 >= 2 && p[i] - 1 > ans) {
                ans = p[i] - 1;
                l = (i - p[i] + 1 + 1) / 2;
            }
        }
        return make_pair(ans, l);
    }
} manacher;

char a[MAX_N];
char c[2];

int main(int argc, char **argv) {
    while (scanf("%s", c) != EOF) {
        int delta = c[0] - 'a';
        scanf("%s", a);
        int len = strlen(a);
        for (int i = 0; i < len; i++) {
            a[i] = (a[i] - 'a' - delta + 26) % 26 + 'a';
        }
        manacher.init(a, len);
        pair<int, int> ans = manacher.max_length();
        if (ans.first < 2) puts("No solution!");
        else {
            printf("%d %d\n", ans.second, ans.second + ans.first - 1);
            for (int i = ans.second, j = 0; j < ans.first; j++) {
                printf("%c", a[i + j]);
            }
            puts("");
        }
    }
    return 0;
}

/**
b babd
a abcd

0 2
aza
No solution!
*/
```

LightOJ 1258
向字符串末尾添加尽量少的任意字符，使得得到的串是一个最长回文子串。
```cpp
#include <cstring>
#include <cstdio>
#include <algorithm>
#include <string>

using namespace std;
typedef long long LL;

const int MAX_N = 1000000 + 16;

struct Manacher {
    static const int $ = -1;
    int b[2 * MAX_N + 1];
    int len_b;
    int p[2 * MAX_N + 1];

    void init(char a[], int n) {
        len_b = 2 * n + 1;
        for (int i = 0; i < n; i++) {
            b[2 * i] = $;
            b[2 * i + 1] = a[i];
        }
        b[len_b - 1] = $;

        int l = 0;
        int r = -1;
        for (int i = 0; i < len_b; i++) {
            if (i > r) {
                p[i] = 1;
            } else {
                p[i] = min(p[l + r - i], r - i + 1);
            }
            while (0 <= i - p[i] && i + p[i] < len_b && b[i - p[i]] == b[i + p[i]]) {
                p[i]++;
            }
            if (i + p[i] - 1 > r) {
                l = i - p[i] + 1;
                r = i + p[i] - 1;
            }
        }
    }

    int get_ans() {
        int ans = 2 * len_b - 1;
        for (int i = len_b / 2; i < len_b; i++) {
            if (i + p[i] - 1 == len_b - 1) {
                int temp = (len_b + (i - len_b / 2) * 2) / 2;
                ans = min(ans, temp);
            }
        }
        return ans;
    }
} manacher;

char a[MAX_N];

int main(int argc, char **argv) {
    int T;
    scanf("%d", &T);
    for (int cas = 1; cas <= T; cas++) {
        scanf("%s", a);
        int len = strlen(a);
        manacher.init(a, len);
        printf("Case %d: %d\n", cas, manacher.get_ans());
    }
    return 0;
}

/**
4
bababababa
pqrs
madamimadam
anncbaaababaaa

Case 1: 11
Case 2: 7
Case 3: 11
Case 4: 19
*/
```