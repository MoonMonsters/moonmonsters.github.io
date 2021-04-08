title: leetcode --- 121. 买卖股票的最佳时机
author: _Tao
tags: []
categories:
  - leetcode
date: 2021-03-27 14:38:00
---
### 题目

[买卖股票的最佳时机](https://leetcode-cn.com/problems/best-time-to-buy-and-sell-stock)

给定一个数组 prices ，它的第 i 个元素 prices[i] 表示一支给定股票第 i 天的价格。

你只能选择 某一天 买入这只股票，并选择在 未来的某一个不同的日子 卖出该股票。设计一个算法来计算你所能获取的最大利润。

返回你可以从这笔交易中获取的最大利润。如果你不能获取任何利润，返回 0 。

 

示例 1：
>输入：[7,1,5,3,6,4] <br/>
输出：5 <br/>
解释：在第 2 天（股票价格 = 1）的时候买入，在第 5 天（股票价格 = 6）的时候卖出，最大利润 = 6-1 = 5 。注意利润不能是 7-1 = 6, 因为卖出价格需要大于买入价格；同时，你不能在买入前卖出股票。<br/>
     
示例 2：
> 输入：prices = [7,6,4,3,1] <br/>
输出：0 <br/>
解释：在这种情况下, 没有交易完成, 所以最大利润为 0。<br/>


### 解法
```python
class Solution:
    def maxProfit(self, prices) -> int:
        if not prices:
            return 0

        minNum = prices[0]
        value = 0
        # 遍历列表, 保存最小值到minNum中
        for n in prices[1:]:
            tmp = n - minNum
            # 判断当前值与最小值的差值
            if tmp > value:
                value = tmp
            if n < minNum:
                minNum = n
        return value


nums = [7, 1, 5, 3, 6, 4]
s = Solution()
print(s.maxProfit(nums))

```

