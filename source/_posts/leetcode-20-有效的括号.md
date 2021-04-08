title: leetcode --- 20.有效的括号
author: _Tao
tags: []
categories:
  - leetcode
date: 2021-03-27 12:55:00
---
### 题目
[有效的括号](https://leetcode-cn.com/problems/valid-parentheses)

给定一个只包括 '('，')'，'{'，'}'，'['，']' 的字符串 s ，判断字符串是否有效。

有效字符串需满足：

左括号必须用相同类型的右括号闭合。
左括号必须以正确的顺序闭合。
 

示例 1：
> 输入：s = "()"
输出：true

示例 2：
>输入：s = "()[]{}"
输出：true

示例 3：
>输入：s = "(]"
输出：false

示例 4：
输入：s = "([)]"
输出：false

示例 5：
> 输入：s = "{[]}"
输出：true


### 解法
```python
class Solution:
    def isValid(self, s: str) -> bool:

        left_dict = {
            "(": 1,
            "[": 2,
            "{": 3
        }
        right_dict = {
            ")": -1,
            "]": -2,
            "}": -3
        }

        stack = list()
        for ss in s:
            if ss in left_dict.keys():
                stack.append(ss)
            else:
                if not stack:
                    return False

                ch = stack.pop(-1)
                if left_dict.get(ch, 99) + right_dict.get(ss, 99) == 0:
                    continue
                else:
                    return False
        if not stack:
            return True

        return False


s = "()[]{}"
so = Solution()
print(so.isValid(s))

```
