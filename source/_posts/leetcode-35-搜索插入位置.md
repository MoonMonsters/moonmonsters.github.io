title: Leetcode --- 35. 搜索插入位置
author: _Tao
tags: []
categories:
  - Leetcode
date: 2021-03-27 13:01:00
---
### 题目

[搜索插入位置](https://leetcode-cn.com/problems/search-insert-position)

给定一个排序数组和一个目标值，在数组中找到目标值，并返回其索引。如果目标值不存在于数组中，返回它将会被按顺序插入的位置。

你可以假设数组中无重复元素。

示例 1:
> 输入: [1,3,5,6], 5 <br/>
输出: 2

示例 2:
> 输入: [1,3,5,6], 2 <br/>
输出: 1

示例 3:
> 输入: [1,3,5,6], 7 <br/>
输出: 4

示例 4:
> 输入: [1,3,5,6], 0 <br/>
输出: 0


### 解法
```python
class Solution:
    def searchInsert(self, nums: list, target: int) -> int:
        for index, value in enumerate(nums):
            if target <= value:
                return index
        else:
            return len(nums)


s = Solution()
ns = [1, 3, 5, 6]
print(s.searchInsert(ns, 5))

```
