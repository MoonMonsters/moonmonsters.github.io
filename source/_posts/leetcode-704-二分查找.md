title: leetcode --- 704. 二分查找
author: _Tao
tags: []
categories:
  - leetcode
date: 2021-03-27 15:10:00
---
### 题目

[二分查找](https://leetcode-cn.com/problems/binary-search)

给定一个 n 个元素有序的（升序）整型数组 nums 和一个目标值 target  ，写一个函数搜索 nums 中的 target，如果目标值存在返回下标，否则返回 -1。


示例
> 输入: nums = [-1,0,3,5,9,12], target = 9 <br/>
输出: 4 <br/>
解释: 9 出现在 nums 中并且下标为 4 <br/>

### 解法
```python
class Solution:
    def search(self, nums: list, target: int) -> int:
        left = 0
        right = len(nums) - 1

        while left <= right:
            mid = (left + right) // 2
            if nums[mid] == target:
                return mid
            elif nums[mid] < target:
                left = mid + 1
            else:
                right = mid - 1

        return -1


s = Solution()
nums = [-1, 0, 3, 5, 9, 12]
target = 2
print(s.search(nums, target))

```
