title: leetcode — 括号生成
author: _Tao
tags: []
categories:
  - leetcode
date: 2020-05-23 17:57:00
---
[括号生成](https://leetcode-cn.com/problems/generate-parentheses/)

给出 n 代表生成括号的对数，请你写出一个函数，使其能够生成所有可能的并且有效的括号组合。

例如，给出 n = 3，生成结果为：
```text
[
  "((()))",
  "(()())",
  "(())()",
  "()(())",
  "()()()"
]
```

<!-- more -->

使用回溯+剪枝做,先往结果中添加左括号,再添加右括号.

```python
class Solution:
	def generateParenthesis(self, n: int) -> list:
		self.__result = []
		self.__add_all(0, 0, n, '')

		return self.__result

	def __add_all(self, used_left, used_right, n, cur):
		"""
		递归计算
		:param used_left: 左括号已使用数量
		:param used_right: 右括号已使用数量
		:param n: 总数
		:param cur: 当前结果
		"""
		# 如果左括号和右括号都已经使用完了,则结束此轮递归
		if used_left == n and used_right == n:
			self.__result.append(cur)
			return

		# 如果左括号还能增加
		if used_left < n:
			self.__add_all(used_left + 1, used_right, n, cur + '(')

		# 如果右括号还能增加,并且右括号的数量要小于左括号的数量
		if used_right < n and used_right < used_left:
			self.__add_all(used_left, used_right + 1, n, cur + ')')


s = Solution()
print(s.generateParenthesis(10))
```