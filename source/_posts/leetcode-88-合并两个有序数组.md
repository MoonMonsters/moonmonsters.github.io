title: Leetcode --- 88. 合并两个有序数组
author: _Tao
tags: []
categories:
  - Leetcode
date: 2021-03-27 14:30:00
---
### 题目

[合并两个有序数组](https://leetcode-cn.com/problems/merge-sorted-array)

给你两个有序整数数组 nums1 和 nums2，请你将 nums2 合并到 nums1 中，使 nums1 成为一个有序数组。

初始化 nums1 和 nums2 的元素数量分别为 m 和 n 。你可以假设 nums1 的空间大小等于 m + n，这样它就有足够的空间保存来自 nums2 的元素。

 

示例 1：
输入：nums1 = [1,2,3,0,0,0], m = 3, nums2 = [2,5,6], n = 3 <br/>
输出：[1,2,2,3,5,6]

示例 2：
输入：nums1 = [1], m = 1, nums2 = [], n = 0 <br/>
输出：[1]


### 解法

```python
class Solution:
    def merge(self, nums1: list, m: int, nums2: list, n: int) -> None:
        """
        Do not return anything, modify nums1 in-place instead.
        """

        tmp = nums1[:m]
        nums1.clear()
        nums1.extend(tmp)
        nums2 = nums2[:n]

        if not nums1:
            nums1.extend(nums2)
            return
        elif not nums2:
            return

        index = 0
        for n2 in nums2:
            while True:
                if n2 <= nums1[index]:
                    nums1.insert(index, n2)
                    break
                elif n2 > nums1[index] and (index == len(nums1) - 1 or n2 < nums1[index + 1]):
                    nums1.insert(index + 1, n2)
                    index += 1
                    break
                else:
                    index += 1

                if index == len(nums1):
                    break


nums1 = [1, 2, 3, 0, 0, 0]
m = 3
nums2 = [2, 5, 6]
n = 3

s = Solution()
s.merge(nums1, m, nums2, n)
print(nums1)

```
