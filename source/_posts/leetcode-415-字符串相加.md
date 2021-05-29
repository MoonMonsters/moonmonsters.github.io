title: Leetcode --- 415. 字符串相加
author: _Tao
tags: []
categories:
  - Leetcode
date: 2021-03-27 15:08:00
---
### 题目

[字符串相加](https://leetcode-cn.com/problems/add-strings/)

给定两个字符串形式的非负整数 num1 和num2 ，计算它们的和。

### 解法
```python
class Solution:
    def addStrings(self, num1: str, num2: str) -> str:
        num1 = num1[::-1]
        num2 = num2[::-1]
        length = min(len(num1), len(num2))
        result = ""
        tmp = 0

        for index in range(length):
            r = int(num1[index]) + int(num2[index]) + tmp
            result += str(r % 10)
            tmp = r // 10

        if length == len(num1):
            num = num2
        else:
            num = num1

        for index in range(length, len(num)):
            r = int(num[index]) + tmp
            result += str(r % 10)
            tmp = r // 10

        if tmp != 0:
            result += str(tmp)

        return result[::-1]


n1 = "123458888"
n2 = "2232342358233289"
s = Solution()
print(s.addStrings(n1, n2))

```