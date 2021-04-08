title: leetcode --- 77. 组合
author: _Tao
tags: []
categories:
  - leetcode
date: 2021-03-27 14:23:00
---
### 题目

[组合](https://leetcode-cn.com/problems/combinations)

给定两个整数 n 和 k，返回 1 ... n 中所有可能的 k 个数的组合。

示例:
> 输入: n = 4, k = 2
输出:
[
  [2,4],
  [3,4],
  [2,3],
  [1,2],
  [1,3],
  [1,4],
]


### 解法
```python
class Solution:

    def combine(self, n: int, k: int):
        import itertools
        nums = range(1, n + 1)
        return list(itertools.combinations(nums, k))


s = Solution()
print(s.combine(4, 2))

```