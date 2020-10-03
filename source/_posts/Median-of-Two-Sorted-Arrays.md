---
title: Median of Two Sorted Arrays
date: 2019-12-29 15:01:00
mathjax: true
tags:
- Algorithm
---

https://leetcode.com/problems/median-of-two-sorted-arrays/

函数$kth$表示归并两个数组时得到的第$k$个元素，函数的计算过程为，每次从$A$或$B$数组的其中一个数组中，找出$p$个元素，这$p$个元素不大于归并得到的第$k$个元素，接着把这$p$个元素排除掉，继续在$A$和$B$数组的剩余部分找归并$A$和$B$数组剩余部分后得到的第$k-p$个元素。
特殊地，当$k=1$或有一个数组为空时可以直接得到答案。
每次找到一个$p \in [1, \lfloor \frac{k}{2} \rfloor]$，假设$A[0:p]$和$B[0:p]$都在归并的结果内，比较$A_p$和$B_p$，如果$A_p < B_p$则说明$A$中前$p$个元素都不大于归并后的第$k$个元素，否则说明$B$中前$p$个元素都不大于归并后的第$k$个元素。

当$p$取$\lfloor \frac{k}{2} \rfloor$时时间复杂度为$O(log(k)) = O(log(n + m))$。

```cpp
class Solution {
public:
    double findMedianSortedArrays(vector<int>& nums1, vector<int>& nums2) {
        int n = nums1.size() + nums2.size();
        if (n % 2 == 1)
            return kth(nums1, 0, nums2, 0, n / 2 + 1);
        else
            return (kth(nums1, 0, nums2, 0, n / 2) + kth(nums1, 0, nums2, 0, n / 2 + 1)) / 2.0;
    }
    int kth(vector<int> &nums1, int n1, vector<int> &nums2, int n2, int k) {
        if (nums1.size() - n1 > nums2.size() - n2)
            return kth(nums2, n2, nums1, n1, k);
        if (nums1.size() - n1 == 0)
            return nums2[n2 + k - 1];
        if (k == 1)
            return min(nums1[n1], nums2[n2]);
        
        int p = min(k / 2, int(nums1.size() - n1));
        if (nums1[n1 + p - 1] < nums2[n2 + p - 1])
            return kth(nums1, n1 + p, nums2, n2, k - p);
        else
            return kth(nums1, n1, nums2, n2 + p, k - p);
    }
};
```