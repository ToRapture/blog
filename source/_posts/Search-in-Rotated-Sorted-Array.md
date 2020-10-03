---
title: Search in Rotated Sorted Array
date: 2020-04-04 11:05:00
tags: Algorithm
---

[33. Search in Rotated Sorted Array](https://leetcode.com/problems/search-in-rotated-sorted-array/)

```cpp
class Solution {
public:
    int search(vector<int>& nums, int target) {
        int l = 0, r = nums.size() - 1;
        while (l <= r) {
            int mid = (l + r) / 2;
            if (nums[mid] == target) return mid;
            if (nums[mid] < target) {
                // target on the left side and mid on the right side
                if (target > nums[r] && nums[mid] < nums[l]) r = mid - 1;
                else l = mid + 1;
            } else {
                // target on the right side and mid on the left side
                if (target < nums[l] && nums[mid] > nums[r]) l = mid + 1;
                else r = mid - 1;
            }
        }
        return -1;
    }
};
```