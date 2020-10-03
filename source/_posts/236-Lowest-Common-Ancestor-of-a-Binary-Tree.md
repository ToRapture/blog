---
title: 236. Lowest Common Ancestor of a Binary Tree
date: 2020-04-10 16:51:00
mathjax: true
tags:
- Algorithm
---

[236. Lowest Common Ancestor of a Binary Tree](https://leetcode.com/problems/lowest-common-ancestor-of-a-binary-tree/)

If we can find two subtrees of root such that both of them has target, then root is the LCA of p and q.
If there is exactly one subtree of root having target and root itself has target, then root is the LCA of p and q too.

```cpp
/**
 * Definition for a binary tree node.
 * struct TreeNode {
 *     int val;
 *     TreeNode *left;
 *     TreeNode *right;
 *     TreeNode(int x) : val(x), left(NULL), right(NULL) {}
 * };
 */
class Solution {
public:
    TreeNode *ans;
    bool hasTarget(TreeNode *root, TreeNode *p, TreeNode *q) {
        if (root == nullptr) return false;
    
        bool left = hasTarget(root->left, p, q);
        bool right = hasTarget(root->right, p, q);
        bool mid = root == p || root == q;
        
        if ((left && right) || (left && mid) || (mid && right)) {
            ans = root;
        }
        
        return left || right || mid;
    }
    
    TreeNode* lowestCommonAncestor(TreeNode* root, TreeNode* p, TreeNode* q) {
        ans = nullptr;
        hasTarget(root, p, q);
        return ans;
    }
};
```