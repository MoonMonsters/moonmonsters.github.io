title: leetcode --- 15.三数之和
author: _Tao
tags: []
categories:
  - leetcode
date: 2021-03-26 00:48:00
---

### 题目

[三数之和](https://leetcode-cn.com/problems/3sum/)

给你一个包含 n 个整数的数组 nums，判断 nums 中是否存在三个元素 a，b，c ，使得 a + b + c = 0 ？请你找出所有和为 0 且不重复的三元组。

注意：答案中不可以包含重复的三元组。

示例 1
> 输入：nums = [-1,0,1,2,-1,-4] <br/>
输出：[[-1,-1,2],[-1,0,1]] <br/>

示例 2
> 输入：nums = [] <br/>
输出：[] <br/>

示例 3
> 输入：nums = [0] <br/>
输出：[] <br/>


### 解析
```python
class Solution:
    def threeSum(self, nums: list) -> list:
        result = set()
        nums = sorted(nums)

        for i in range(len(nums)):

            m = nums[i]
            if m > 0:
                break

            left = i + 1
            right = len(nums) - 1
            while left < right:
                n = nums[left]
                o = nums[right]
                if m + n + o == 0:
                    result.add(tuple(sorted([m, n, o])))
                    left += 1
                    right -= 1
                elif m + n + o > 0:
                    right -= 1
                else:
                    left += 1

        return [list(r) for r in result]


s = Solution()
ns = [-1,0,1,2,-1,-4]
print(s.threeSum(ns))

```