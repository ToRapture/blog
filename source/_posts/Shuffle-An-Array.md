---
title: Shuffle An Array
date: 2020-11-01 22:32:44
tags:
- Algorithm
---

[384. Shuffle an Array](https://leetcode.com/problems/shuffle-an-array/)

```python
import copy
import random


class Solution(object):
    def __init__(self, nums):
        self.ori = copy.copy(nums)
        self.nums = nums

    def reset(self):
        return self.ori

    def shuffle(self):
        for i in range(len(self.nums)):
            pos = random.randrange(i, len(self.nums))
            self.nums[i], self.nums[pos] = self.nums[pos], self.nums[i]
        return self.nums
```