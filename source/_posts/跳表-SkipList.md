---
title: 跳表 SkipList
date: 2019-11-20 16:25:00
tags:
- 数据结构
---

跳跃表实现简单，空间复杂度和时间复杂度也较好，Redis中使用跳表而不是红黑树。

* * *

实现参考了：
- [跳跃列表 - 维基百科，自由的百科全书](https://zh.wikipedia.org/wiki/%E8%B7%B3%E8%B7%83%E5%88%97%E8%A1%A8)
- 《Redis设计与实现》第五章 跳跃表
- Redis源码3.0分支src/t_zset.c等文件

* * *

插入时的核心逻辑：
1. 找到插入的位置
2. 随机得到新插入节点的level
3. 处理为了插入当前节点穿过的指针和未穿过的指针的指向和跨度

删除时的核心逻辑：
1. 找到删除的位置
2. 处理要删除的节点穿过的指针和未穿过的指针的指向和跨度
3. 如果可以，减小跳跃表的level

* * *

下面两个题可以使用平衡树，这里为了练习使用跳跃表，注意根据每个题的题意特殊处理。
[SPOJ ORDERSET](https://www.spoj.com/problems/ORDERSET/)
```cpp
#include <bits/stdc++.h>
#define DBG(x) cerr << #x << " = " << x << endl

using namespace std;
typedef long long LL;

const int MAX_LEVEL = 32;
const int MAX_N = 200000;

struct ListNode {
    struct Forward {
        int span;
        ListNode *forward;
    };
    int val;
    vector<Forward> levels;
};

namespace memory {
    ListNode nodes[MAX_N + 1 + 16];
    int cnt;

    void init() {
        cnt = 0;
    }

    ListNode *new_node(int level, int val) {
        nodes[cnt].levels.resize(level);
        for (int i = 0; i < level; i++) {
            nodes[cnt].levels[i].forward = nullptr;
            nodes[cnt].levels[i].span = 0;
        }
        nodes[cnt].val = val;
        return &nodes[cnt++];
    }
}

struct SkipList {
    ListNode *header;
    int level;
    int length;

    void init() {
        memory::init();
        srand(time(0));

        level = 1;
        length = 0;
        header = memory::new_node(MAX_LEVEL, 0);
    }

    int get_level() {
        int level = 1;
        while (rand() % 2 && level + 1 < MAX_LEVEL)
            level++;
        return level;
    }

    void insert(int x) {
        ListNode *update[MAX_LEVEL];
        int rank[MAX_LEVEL];

        ListNode *p = header;
        for (int i = level - 1; i >= 0; i--) {
            rank[i] = i == (level - 1) ? 0 : rank[i + 1];
            while (p->levels[i].forward && p->levels[i].forward->val <= x) {
                rank[i] += p->levels[i].span;
                p = p->levels[i].forward;
            }
            update[i] = p;
        }
        if (p != header && p->val == x)
            return;

        int level_of_node = get_level();
        if (level_of_node > level) {
            for (int i = level; i < level_of_node; i++) {
                header->levels[i].span = length;
                update[i] = header;
                rank[i] = 0;
            }
            level = level_of_node;
        }
        ListNode *node = memory::new_node(level_of_node, x);
        for (int i = 0; i < level_of_node; i++) {
            node->levels[i].forward = update[i]->levels[i].forward;
            node->levels[i].span = update[i]->levels[i].span - (rank[0] - rank[i]);

            update[i]->levels[i].forward = node;
            update[i]->levels[i].span = rank[0] - rank[i] + 1;
        }

        for (int i = level_of_node; i < level; i++) {
            update[i]->levels[i].span++;
        }

        length++;
    }

    void remove(int x) {
        ListNode *update[MAX_LEVEL];

        ListNode *p = header;
        for (int i = level - 1; i >= 0; i--) {
            while (p->levels[i].forward && p->levels[i].forward->val < x) {
                p = p->levels[i].forward;
            }
            update[i] = p;
        }

        p = p->levels[0].forward;
        if (!p || p->val != x)
            return;

        for (int i = level - 1; i >= 0; i--) {
            if (update[i]->levels[i].forward == p) {
                update[i]->levels[i].forward = p->levels[i].forward;
                update[i]->levels[i].span += p->levels[i].span - 1;
            } else {
                update[i]->levels[i].span--;
            }
        }

        while (level > 1 && !header->levels[level - 1].forward)
            level--;
        length--;
    }

    int kth(int k) {
        int span = 0;
        ListNode *p = header;
        for (int i = level - 1; i >= 0; i--) {
            while (p->levels[i].forward && span + p->levels[i].span <= k) {
                span += p->levels[i].span;
                p = p->levels[i].forward;
            }
        }
        return p->val;
    }

    int rank(int x) {
        int ans = 0;
        ListNode *p = header;
        for (int i = level - 1; i >= 0; i--) {
            while (p->levels[i].forward && p->levels[i].forward->val < x) {
                ans += p->levels[i].span;
                p = p->levels[i].forward;
            }
        }
        return ans;
    }

} skip_list;

int main(int argc, char **argv) {
    int n;
    while (scanf("%d", &n) != EOF) {
        skip_list.init();
        for (int i = 0; i < n; i++) {
            char op[2];
            int x;
            scanf("%s%d", op, &x);
            if (op[0] == 'I') {
                skip_list.insert(x);
            } else if (op[0] == 'D') {
                skip_list.remove(x);
            } else if (op[0] == 'K') {
                if (x > skip_list.length) {
                    puts("invalid");
                } else {
                    printf("%d\n", skip_list.kth(x));
                };
            } else if (op[0] == 'C') {
                printf("%d\n", skip_list.rank(x));
            }
        }
    }
    return 0;
}

/**
8
I -1
I -1
I 2
C 0
K 2
D -1
K 1
K 2

1
2
2
invalid
*/
```

[HDU 4585](http://acm.hdu.edu.cn/showproblem.php?pid=4585)
```cpp
#include <bits/stdc++.h>
#define DBG(x) cerr << #x << " = " << x << endl

using namespace std;
typedef long long LL;

const int MAX_LEVEL = 32;
const int MAX_N = 100000;

struct ListNode {
    struct Forward {
        int span;
        ListNode *forward;
    };
    pair<int, int> val;
    vector<Forward> levels;
};

namespace memory {
    ListNode nodes[MAX_N + 1 + 16];
    int cnt;

    void init() {
        cnt = 0;
    }

    ListNode *new_node(int level, pair<int, int> val) {
        nodes[cnt].levels.resize(level);
        for (int i = 0; i < level; i++) {
            nodes[cnt].levels[i].forward = nullptr;
            nodes[cnt].levels[i].span = 0;
        }
        nodes[cnt].val = val;
        return &nodes[cnt++];
    }
}

struct SkipList {
    ListNode *header;
    int level;
    int length;

    void init() {
        memory::init();
        srand(time(0));

        level = 1;
        length = 0;
        header = memory::new_node(MAX_LEVEL, make_pair(0, 0));
    }

    int get_level() {
        int level = 1;
        while (rand() % 2 && level + 1 < MAX_LEVEL)
            level++;
        return level;
    }

    void insert(pair<int, int> x) {
        ListNode *update[MAX_LEVEL];
        int rank[MAX_LEVEL];

        ListNode *p = header;
        for (int i = level - 1; i >= 0; i--) {
            rank[i] = i == (level - 1) ? 0 : rank[i + 1];
            while (p->levels[i].forward && p->levels[i].forward->val <= x) {
                rank[i] += p->levels[i].span;
                p = p->levels[i].forward;
            }
            update[i] = p;
        }
        if (p != header && p->val == x)
            return;

        int level_of_node = get_level();
        if (level_of_node > level) {
            for (int i = level; i < level_of_node; i++) {
                header->levels[i].span = length;
                update[i] = header;
                rank[i] = 0;
            }
            level = level_of_node;
        }
        ListNode *node = memory::new_node(level_of_node, x);
        for (int i = 0; i < level_of_node; i++) {
            node->levels[i].forward = update[i]->levels[i].forward;
            node->levels[i].span = update[i]->levels[i].span - (rank[0] - rank[i]);

            update[i]->levels[i].forward = node;
            update[i]->levels[i].span = rank[0] - rank[i] + 1;
        }

        for (int i = level_of_node; i < level; i++) {
            update[i]->levels[i].span++;
        }

        length++;
    }

    pair<int, int> find(pair<int, int> x) {
        ListNode *p = header;
        for (int i = level - 1; i >= 0; i--) {
            while (p->levels[i].forward && p->levels[i].forward->val < x)
                p = p->levels[i].forward;
        }

        if (p == header) {
            p = p->levels[0].forward;
            return p->val;
        }

        if (p->levels[0].forward && p->levels[0].forward->val.first - x.first < x.first - p->val.first) {
            p = p->levels[0].forward;
        }
        return p->val;
    }
} skip_list;

int main(int argc, char **argv) {
    int n;
    while (true) {
        scanf("%d", &n);
        if (n == 0) break;
        skip_list.init();
        skip_list.insert(make_pair(1000000000, 1));
        for (int i = 0; i < n; i++) {
            int k, g;
            scanf("%d%d", &k, &g);
            printf("%d %d\n", k, skip_list.find(make_pair(g, k)).second);
            skip_list.insert(make_pair(g, k));
        }
    }
    return 0;
}

/**
3
2 1
3 3
4 2
0

2 1
3 2
4 2
*/
```