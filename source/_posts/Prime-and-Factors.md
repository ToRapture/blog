---
title: Prime and Factors
date: 2020-04-13 14:25:00
mathjax: true
tags:
- Algorithm
---

[HDU 1164](http://acm.hdu.edu.cn/showproblem.php?pid=1164)
```cpp
#include <bits/stdc++.h>
#define DBG(x) cerr << #x << " = " << x << endl

using namespace std;
typedef long long LL;

const int MAXN = 65535 + 16;
bool is_prime[MAXN];
vector<int> primes;

struct Factor {
    int value;
    int cnt;

    Factor() {}

    Factor(int value, int cnt) : value(value), cnt(cnt) {}
};

void init_prime() {
    fill(is_prime, is_prime + MAXN, true);
    is_prime[0] = is_prime[1] = false;
    for (int i = 2; i < MAXN; i++) {
        if (!is_prime[i]) continue;
        primes.push_back(i);
        for (long long j = 1LL * i * i; j < MAXN; j += i) {
            is_prime[j] = false;
        }
    }
}

vector<Factor> get_factors(int x) {
    vector<Factor> factors;
    int temp = x;
    for (int i = 0; i < primes.size() && primes[i] <= x / primes[i]; i++) {
        if (temp % primes[i] == 0) {
            Factor factor(primes[i], 0);
            while (temp % primes[i] == 0) {
                factor.cnt++;
                temp /= primes[i];
            }
            factors.push_back(factor);
        }
    }

    if (temp > 1) {
        factors.push_back(Factor(temp, 1));
    }

    return factors;
}

int main(int argc, char **argv) {
    init_prime();
    int x;
    while (scanf("%d", &x) != EOF) {
        vector<Factor> factors = get_factors(x);
        string ans;
        for (int i = 0; i < factors.size(); i++) {
            for (int j = 0; j < factors[i].cnt; j++) {
                if (!ans.empty()) ans += '*';
                ans += to_string(factors[i].value);
            }
        }
        puts(ans.c_str());
    }
    return 0;
}
```

[HDU 4548](http://acm.hdu.edu.cn/showproblem.php?pid=4548)
```cpp
#include <bits/stdc++.h>
#define DBG(x) cerr << #x << " = " << x << endl

using namespace std;
typedef long long LL;

const int MAXN = 1000000 + 16;

struct Prime {
    bool is_prime[MAXN];
    int sum[MAXN];

    Prime() {
        fill(is_prime, is_prime + MAXN, true);
        fill(sum, sum + MAXN, 0);
        is_prime[0] = is_prime[1] = false;
        for (int i = 2; i < MAXN; i++) {
            if (!is_prime[i]) continue;
            for (long long j = 1LL * i * i; j < MAXN; j += i) {
                is_prime[j] = false;
            }
        }
        for (int i = 1; i < MAXN; i++) {
            sum[i] = sum[i - 1];
            if (is_beautiful(i)) sum[i]++;
        }
    }

    bool is_beautiful(int x) {
        if (!is_prime[x]) return false;
        int s = 0;
        while (x) {
            s += x % 10;
            x /= 10;
        }
        return is_prime[s];
    }

    int get(int l, int r) {
        return sum[r] - sum[l - 1];
    }
} p;

int main(int argc, char **argv) {
    int T;
    scanf("%d", &T);
    for (int cas = 1; cas <= T; cas++) {
        int l, r;
        scanf("%d%d", &l, &r);
        printf("Case #%d: %d\n", cas, p.get(l, r));
    }
    return 0;
}
```