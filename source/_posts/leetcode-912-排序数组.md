title: Leetcode --- 912. 排序数组
author: _Tao
tags: []
categories:
  - Leetcode
date: 2021-03-27 15:12:00
---
### 题目

[排序数组](https://leetcode-cn.com/problems/sort-an-array/)

示例
> 输入：nums = [5,2,3,1] <br/>
输出：[1,2,3,5]

 
### 解法
```python
class Solution:
    def sortArray(self, nums: list) -> list:
        ns = nums[:]
        return self._quick_sort(nums, 0, len(ns) - 1)

    def _quick_sort(self, nums, left, right):

        if left >= right:
            return nums

        base_index = self._sort(left, right, nums)
        self._quick_sort(nums, left, base_index - 1)
        self._quick_sort(nums, base_index + 1, right)

        return nums

    def _sort(self, left, right, nums):

        temp = left
        base = nums[left]
        while left != right:
            while left < right and nums[right] >= base:
                right -= 1
            while left < right and nums[left] <= base:
                left += 1

            if left < right:
                nums[left], nums[right] = nums[right], nums[left]

        nums[temp] = nums[left]
        nums[left] = base
        return left


num = [5, 1, 1, 2, 0, 0]
s = Solution()
print(s.sortArray(num))

```