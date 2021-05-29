title: Leetcode --- 寻找两个有序数组的中位数
author: _Tao
tags: []
categories:
  - Leetcode
  - ''
date: 2020-05-23 17:50:00
---
给定两个大小为 m 和 n 的有序数组 nums1 和 nums2。
请你找出这两个有序数组的中位数，并且要求算法的时间复杂度为 O(log(m + n))。
你可以假设 nums1 和 nums2 不会同时为空。

nums1 = [1, 3] 
nums2 = [2]

则中位数是 2.0

<!-- more -->

比较简单把，两个排序好的数组，用一个循环就能搞定了.

```python
# !/usr/bin/env python
# -*- coding:utf-8 -*-
# __author__ = 'ChenTao'


class Solution:
    def findMedianSortedArrays(self, nums1: list, nums2: list) -> float:
        result = []
        m, n = 0, 0
        while m != len(nums1) and n != len(nums2):
            if nums1[m] < nums2[n]:
                result.append(nums1[m])
                m += 1
            else:
                result.append(nums2[n])
                n += 1

        result.extend(nums1[m:])
        result.extend(nums2[n:])
        print(result)
        if len(result) % 2 == 0:
            mid = int(len(result) / 2)
            return (result[mid - 1] + result[mid]) / 2.0
        else:
            mid = int((len(result) - 1) / 2)
            return result[mid] * 1.0


s = Solution()
nums1 = []
nums2 = [1]
print(s.findMedianSortedArrays(nums1, nums2))
```