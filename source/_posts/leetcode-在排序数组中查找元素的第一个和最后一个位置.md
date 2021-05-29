title: Leetcode --- 在排序数组中查找元素的第一个和最后一个位置
author: _Tao
tags: []
categories:
  - Leetcode
date: 2020-05-23 18:58:00
---
### 题目

[在排序数组中查找元素的第一个和最后一个位置](https://leetcode-cn.com/problems/find-first-and-last-position-of-element-in-sorted-array/)


给定一个按照升序排列的整数数组 nums，和一个目标值 target。找出给定目标值在数组中的开始位置和结束位置。
你的算法时间复杂度必须是 O(log n) 级别。
如果数组中不存在目标值，返回 [-1, -1]。

示例1:
>输入: nums = [5,7,7,8,8,10], target = 8
输出: [3,4]

示例2:
>输入: nums = [5,7,7,8,8,10], target = 6
输出: [-1,-1]

<!-- more -->

### 代码

题目很简单，二分法解决。
```python
# !/usr/bin/env python
# -*- coding:utf-8 -*-
# __author__ = 'ChenTao'


class Solution:
    def searchRange(self, nums: list, target: int) -> list:

        if not nums:
            return [-1, -1]

        left = 0
        right = len(nums) - 1
        min_index = -1
        max_index = -1
        # 找到某个位置的值，不管是第几个
        cur_index = -1
        # 二分查找
        while left <= right:
            mid = int((left + right) / 2)
            if nums[mid] == target:
                cur_index = mid
                break
            elif nums[mid] < target:
                left = mid + 1
            else:
                right = mid - 1

        # 采用的是顺序遍历，会更加耗时
        # try:
        #     cur_index = nums.index(target)
        # except:
        #     cur_index = -1

        # 从找到的值的位置往前遍历，找到最小值
        for index in range(cur_index, -1, -1):
            if nums[index] == target:
                min_index = index
            else:
                break

        # 往后遍历找到最大值
        for index in range(cur_index, len(nums), 1):
            if nums[index] == target:
                max_index = index
            else:
                break

        # 如果存在值，不能返回[-1, -1]
        if min_index == -1 or max_index == -1:
            min_index = max_index = max_index if max_index > min_index else min_index

        return [min_index, max_index]


so = Solution()

nums = [5, 7, 7, 8, 8, 10]
# nums = [1, 4]
target = 8
print(so.searchRange(nums, target))

```

然后在题解区看到了另一种解法，我一直认为 in 关键字和 list的index都是顺序遍历，为什么复杂度反而不会增加很多呢？相比于我用二分法解，时间复杂度和空间复杂度几乎没什么增加的。

```python
class Solution(object):
    def searchRange(self, nums, target):
        """
        :type nums: List[int]
        :type target: int
        :rtype: List[int]
        """
        if target in nums:
            start = nums.index(target)
            end = len(nums) - nums[::-1].index(target) - 1
            return [start,end]
        else:
            return [-1,-1]

```