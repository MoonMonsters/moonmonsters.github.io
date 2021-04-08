title: leetcode --- 53. 最大子序和
author: _Tao
tags: []
categories:
  - leetcode
date: 2021-03-27 14:05:00
---
### 题目

[最大子序和](https://leetcode-cn.com/problems/maximum-subarray)

给定一个整数数组 nums ，找到一个具有最大和的连续子数组（子数组最少包含一个元素），返回其最大和。

 
示例 1：
> 输入：nums = [-2,1,-3,4,-1,2,1,-5,4]
输出：6
解释：连续子数组 [4,-1,2,1] 的和最大，为 6 。

> 示例 2：
输入：nums = [1]
输出：1

> 示例 3：
输入：nums = [0]
输出：0

> 示例 4：
输入：nums = [-1]
输出：-1

示例 5：
> 输入：nums = [-100000]
输出：-100000

### 解法
```python
class Solution:
    def maxSubArray(self, nums: list) -> int:

        total = nums[0]
        temp = nums[0]

        for n in nums[1:]:
            if temp < 0:
                temp = 0
            temp += n
            if temp > total:
                total = temp

        return total


nums = [-2, 1, -3, 4, -1, 2, 1, -5, 4]
s = Solution()
print(s.maxSubArray(nums))

```