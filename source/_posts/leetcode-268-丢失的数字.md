title: Leetcode --- 268. 丢失的数字
author: _Tao
tags: []
categories:
  - Leetcode
date: 2021-03-27 15:04:00
---
### 题目

[丢失的数字](https://leetcode-cn.com/problems/missing-number)


给定一个包含 [0, n] 中 n 个数的数组 nums ，找出 [0, n] 这个范围内没有出现在数组中的那个数。

示例 
>输入：nums = [3,0,1] <br/>
输出：2 <br/>
解释：n = 3，因为有 3 个数字，所以所有的数字都在范围 [0,3] 内。2 是丢失的数字，因为它没有出现在 nums 中。<br/>

### 解法
```python
class Solution(object):
    def missingNumber(self, nums):
        """
        :type nums: List[int]
        :rtype: int
        """
        n = len(nums)
        if n % 2 != 0:
            n = n + 1
        # 计算该数字如果存在的总和
        total = (1 + n) * (n // 2)
        if n != len(nums):
            total -= n
        # 完整数组的总和 - 当前总和
        return total - sum(nums)


s = Solution()
nums = [0]
print(s.missingNumber(nums))

```
