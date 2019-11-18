---
layout:     post
title:      Leetcode 238
subtitle:   Product of Array Except Self
date:       2019-11-18
author:     Song
header-img: img/post-bg-avacado1.jpg
catalog: true
tags:
    - Leetcode
    - Algorithm
---

# Introduction
Given an array nums of n integers where n > 1, return an array output such that output[i] is equal to the product of all the elements of nums except nums[i].

**Example:**

Input: [1, 2, 3, 4]

Output: [24,12,8,6]

**Note:**
Please solve it without division and in O(n).

**Follow up:**
Could you solve it with constant space complexity? (The output array does not count as extra space for the purpose of space complexity analysis.)

# My Solution

## O(N) Space Complexity
```python
class Solution:
    def productExceptSelf(self, nums: List[int]) -> List[int]:
        left, right = [1], [1]
        ans = []
        for i in range(len(nums)):
            left.append(left[i]*nums[i])
            right.append(right[i]*nums[len(nums)-i-1])
        for i in range(len(nums)):
            ans.append(left[i]*right[len(nums)-i-1])
        return ans
```
## O(1) Space Complexity
```python
class Solution:
    def productExceptSelf(self, nums: List[int]) -> List[int]:
        ans = [1]
        for i in range(len(nums)-1):
            ans.append(ans[i]*nums[i])
        tmp = 1
        for i in range(len(nums)-1, -1, -1):
            ans[i] *= tmp
            tmp *= nums[i]
        return ans
```
