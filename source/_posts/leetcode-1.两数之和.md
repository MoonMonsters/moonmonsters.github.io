title: Leetcode --- 1.两数之和
author: _Tao
tags: []
categories:
  - Leetcode
date: 2021-03-26 00:39:00
---

### 题目

[两数之和](https://leetcode-cn.com/problems/two-sum/)


给定一个整数数组 nums 和一个整数目标值 target，请你在该数组中找出 和为目标值 的那 两个 整数，并返回它们的数组下标。

你可以假设每种输入只会对应一个答案。但是，数组中同一个元素在答案里不能重复出现。

你可以按任意顺序返回答案。

示例1
> 输入：nums = [2,7,11,15], target = 9 <br/>
输出：[0,1] <br/>
解释：因为 nums[0] + nums[1] == 9 ，返回 [0, 1] 。<br/>

示例 2：
> 输入：nums = [3,2,4], target = 6 <br/>
输出：[1,2]	<br/>

示例 3：
> 输入：nums = [3,3], target = 6	<br/>
输出：[0,1]	<br/>


### 解法
```python
class Solution:
    def twoSum(self, nums: list, target: int) -> list:
        numDict = dict()
        for index, n in enumerate(nums):
            numDict.setdefault(n, []).append(index)

        for n in nums:
            tmp = target - n
            values = numDict.get(tmp)
            if values:
                if tmp == n:
                    if len(values) >= 2:
                        return values[:2]
                else:
                    return [numDict[n][0], numDict[tmp][0]]


ns = [3, 3]
target = 6
s = Solution()
print(s.twoSum(ns, target))

```

