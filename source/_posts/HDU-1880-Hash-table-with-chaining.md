---
title: HDU 1880 Hash table with chaining
date: 2019-11-21 10:40:00
tags: Algorithm
---

[HDU 1880](http://acm.hdu.edu.cn/showproblem.php?pid=1880)
```cpp
#include <bits/stdc++.h>
#define DBG(x) cerr << #x << " = " << x << endl

using namespace std;
typedef long long LL;

const unsigned int KEY = 137;
const int BUCKET_NUM = 120000;

struct Hash {
    vector<pair<string, string>> h[BUCKET_NUM];

    unsigned int get_hash(const string &s) {
        unsigned int hs = 0;
        for (auto c : s)
            hs = (hs * KEY + c) % BUCKET_NUM;
        return hs;
    }

    void insert(const string &k, const string &v) {
        unsigned int hs = get_hash(k);
        h[hs].push_back(make_pair(k, v));
    }

    string *get(const string &key) {
        unsigned int hs = get_hash(key);
        for (auto &kv : h[hs]) {
            if (kv.first == key)
                return &kv.second;
        }
        return nullptr;
    }
} key2val, val2key;

pair<string, string> get_kv(const string &str) {
    unsigned long pos = 0;
    for (int i = 0; i < str.size(); i++) {
        if (str[i] == ']') {
            pos = i;
            break;
        }
    }
    auto k = str.substr(1, pos - 1);
    auto v = str.substr(pos + 2, str.size() - pos - 2);
    return make_pair(k, v);
}

int main(int argc, char **argv) {
    ios::sync_with_stdio(false);
    string s;
    while (true) {
        getline(cin, s);
        if (s == "@END@") break;
        auto kv = get_kv(s);
        key2val.insert(kv.first, kv.second);
        val2key.insert(kv.second, kv.first);
    }
    int n;
    cin >> n;
    cin.get();
    for (int i = 0; i < n; i++) {
        getline(cin, s);
        if (s[0] == '[') {
            auto val = key2val.get(s.substr(1, s.size() - 2));
            if (val == nullptr) {
                cout << "what?" << "\n";
            } else {
                cout << *val << "\n";
            }
        } else {
            auto key = val2key.get(s);
            if (key == nullptr) {
                cout << "what?" << "\n";
            } else {
                cout << *key << "\n";
            }
        }
    }
    return 0;
}

/**
[expelliarmus] the disarming charm
[rictusempra] send a jet of silver light to hit the enemy
[tarantallegra] control the movement of one's legs
[serpensortia] shoot a snake out of the end of one's wand
[lumos] light the wand
[obliviate] the memory charm
[expecto patronum] send a Patronus to the dementors
[accio] the summoning charm
@END@
4
[lumos]
the summoning charm
[arha]
take me to the sky

light the wand
accio
what?
what?
*/
```