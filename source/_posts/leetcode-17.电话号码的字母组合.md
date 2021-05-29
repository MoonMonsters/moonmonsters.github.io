title: Leetcode --- 17.电话号码的字母组合
author: _Tao
tags: []
categories:
  - Leetcode
date: 2021-03-26 00:51:00
---

### 题目

[电话号码的字母组合](https://leetcode-cn.com/problems/letter-combinations-of-a-phone-number/)

给定一个仅包含数字 2-9 的字符串，返回所有它能表示的字母组合。答案可以按 任意顺序 返回。

给出数字到字母的映射如下（与电话按键相同）。注意 1 不对应任何字母。
![](https://qxinhai.oss-cn-shenzhen.aliyuncs.com/hexo/20210326005238.png)

示例 1：
> 输入：digits = "23" <br/>
输出：["ad","ae","af","bd","be","bf","cd","ce","cf"] <br/>

示例 2：
> 输入：digits = "" <br/>
输出：[]

示例 3：
> 输入：digits = "2" <br/>
输出：["a","b","c"]


### 解法
```python
class Solution:
    def letterCombinations(self, digits: str):
        number = {
            "2": ["a", "b", "c"],
            "3": ["d", "e", "f"],
            "4": ["g", "h", "i"],
            "5": ["j", "k", "l"],
            "6": ["m", "n", "o"],
            "7": ["p", "q", "r", "s"],
            "8": ["t", "u", "v"],
            "9": ["w", "x", "y", "z"]
        }

        if not digits or digits == "1":
            return []
        ret = [""]
        for i in digits:
            ret = [r + n for r in ret for n in number[i]]

        return ret


s = Solution()
d = "23"
print(s.letterCombinations(d))

```
