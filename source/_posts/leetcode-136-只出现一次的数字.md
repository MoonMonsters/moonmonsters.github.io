title: leetcode --- 136. 只出现一次的数字
author: _Tao
tags: []
categories:
  - leetcode
date: 2021-03-27 14:40:00
---
### 题目

[只出现一次的数字](https://leetcode-cn.com/problems/single-number)

给定一个非空整数数组，除了某个元素只出现一次以外，其余每个元素均出现两次。找出那个只出现了一次的元素。

说明：

你的算法应该具有线性时间复杂度。 你可以不使用额外空间来实现吗？

示例 1:
>输入: [2,2,1] <br/>
输出: 1

示例 2:
> 输入: [4,1,2,1,2] <br/>
输出: 4


### 解法
```python
class Solution:
    def singleNumber(self, nums: list) -> int:
        t = nums[0]
        for n in nums[1:]:
            t ^= n
        return t


s = Solution()
ns = [4, 1, 2, 1, 2]
print(s.singleNumber(ns))
```