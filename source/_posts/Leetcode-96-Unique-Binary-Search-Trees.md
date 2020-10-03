---
title: Leetcode 96. Unique Binary Search Trees
date: 2020-04-07 20:02:00
tags:
- Algorithm
---

[Unique Binary Search Trees](https://leetcode.com/problems/unique-binary-search-trees/)

```cpp
class Solution {
public:
    int numTrees(int n) {
        // We consider the position of node n in the tree first.
        // Because all nodes on the tree except for node n is less than node,
        // if node n has children, that must be its left child.
        // Node n can have a parent if the value of its parent is less than its.
        // So we separate the n - 1 nodes into 2 parts, the first part is considered to be the parent part of node n,
        // and the second part is considered to be the left child part of node n.
        // We traverse every possible separation, for each separation we add the multiplication of the number of the combination
        // of the two parts to the answer of node n.
        vector<int> dp(n + 1, 0);
        dp[0] = 1;
        for (int i = 1; i <= n; i++) {
            for (int j = 0; j < i; j++) {
                dp[i] += dp[j] * dp[i - 1 - j];
            }
        }
        return dp[n];
    }
};
```