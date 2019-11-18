---
layout:     post
title:      Leetcode 238
subtitle:   Product of Array Except Self
date:       2019-11-18
author:     Song
header-img: img/post_bkg_avacado1.jpg
catalog: true
tags:
    - Leetcode
    - Algorithm
---

# Description
Given an array nums of n integers where n > 1, return an array output such that output[i] is equal to the product of all the elements of nums except nums[i].

**Example:**

Input: [1, 2, 3, 4]<br>
Output: [24,12,8,6]

**Note:**
Please solve it without division and in O(n).

**Follow up:**
Could you solve it with constant space complexity? (The output array does not count as extra space for the purpose of space complexity analysis.)

# My Solution

## Solution1: 
**O(N) Time COmplexity, O(N) Space Complexity**

每一个数的对应的乘积其实可以表示为该数左边的所有数的乘积乘以该数右边的所有数的乘积，因此最简单的方法就是先用两个列表从左到右和从右到左的连续乘积存出来，类似于numpy cumprod用法。
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
## Solution2:
**O(N) Time COmplexity, O(1) Space Complexity**

一个取巧的方法，先存一个从左到右的连续乘积，然后再从右到左的乘回去。
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
