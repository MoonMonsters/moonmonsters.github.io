title: Leetcode --- 最接近的三数之和
author: _Tao
tags: []
categories:
  - Leetcode
date: 2020-05-23 19:01:00
---
### 题目

[最接近的三数之和](https://leetcode-cn.com/problems/3sum-closest/)<br/>
给定一个包括 n 个整数的数组 nums 和 一个目标值 target。找出 nums 中的三个整数，使得它们的和与 target 最接近。返回这三个数的和。假定每组输入只存在唯一答案。
>
例如，给定数组 nums = [-1，2，1，-4], 和 target = 1. 与 target 最接近的三个数的和为 2. (-1 + 2 + 1 = 2).

<!-- more -->

### 题解

```python
# !/usr/bin/env python
# -*- coding:utf-8 -*-
# __author__ = 'ChenTao'

import sys


class Solution:
    def threeSumClosest(self, nums: list, target: int) -> int:
        # 排序
        nums.sort()
        # 最大偏差
        tmp = [sys.maxsize, sys.maxsize, sys.maxsize]
        for first in range(len(nums) - 2):
            second = first + 1
            # 第三个数从最右边开始
            third = len(nums) - 1

            # 保证有效范围
            while second < third < len(nums):
                x = nums[first] + nums[second] + nums[third]

                if x == target:
                    tmp = [nums[first], nums[second], nums[third]]
                    break
                # 计算差值的绝对值
                elif abs(x - target) < abs(sum(tmp) - target):
                    tmp = [nums[first], nums[second], nums[third]]

                # 挪动
                if x < target:
                    second += 1
                else:
                    third -= 1

        return sum(tmp)


s = Solution()
# nums = [0, 2, 1, -3]
# nums = [1, 1, -1, -1, 3]
nums = [1, 2, 4, 8, 16, 32, 64, 128]
target = 82
print(s.threeSumClosest(nums, target))

```