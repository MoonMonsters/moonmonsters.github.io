title: leetcode --- 169. 多数元素
author: _Tao
tags: []
categories:
  - leetcode
date: 2021-03-27 14:52:00
---
### 题目

[多数元素](https://leetcode-cn.com/problems/majority-element)

给定一个大小为 n 的数组，找到其中的多数元素。多数元素是指在数组中出现次数 大于 ⌊ n/2 ⌋ 的元素。

你可以假设数组是非空的，并且给定的数组总是存在多数元素。


示例 1：
> 输入：[3,2,3]
输出：3

示例 2：
> 输入：[2,2,1,1,1,2,2]
输出：2


### 解法
```python
class Solution:
    def majorityElement(self, nums: list) -> int:
        nDict = dict()
        half = len(nums) / 2
        for n in nums:
            d = nDict.get(n, 0)
            d += 1
            if d >= half:
                return n
            nDict[n] = d

        return -1


nums = [3, 2, 3]
s = Solution()
print(s.majorityElement(nums))

```