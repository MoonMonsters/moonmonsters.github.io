title: leetcode --- 电话号码的字母组合
author: _Tao
tags: []
categories:
  - leetcode
date: 2020-05-23 17:59:00
---
[电话号码的字母组合](https://leetcode-cn.com/problems/letter-combinations-of-a-phone-number/)

给定一个仅包含数字 2-9 的字符串，返回所有它能表示的字母组合。

给出数字到字母的映射如下（与电话按键相同）。注意 1 不对应任何字母。

![](https://qxinhai.oss-cn-shenzhen.aliyuncs.com/pic/leetcode-17.png)
示例:
```text
输入："23"
输出：["ad", "ae", "af", "bd", "be", "bf", "cd", "ce", "cf"].
```

<!-- more -->

用递归搞定,不难,解释看代码就行.

```python
class Solution:
	def letterCombinations(self, digits: str) -> list:
		# 字母键盘对应的字符
		self.__values = [
			'', '',
			'abc', 'def',
			'ghi', 'jkl', 'mno',
			'qprs', 'tuv', 'wxyz'
		]

		# 处理输入""的特殊情况
		if not digits:
			return []

		self.__result = []
		self.__counter(digits, 0, '')

		return self.__result

	def __counter(self, digits, index, cur):
		# 处理前n-1个字符,即,如果是234,就处理23两个数字
		if index < len(digits) - 1:
			# index指的是当前digits中的第几个位置的数字
			# 将对应index的键盘的字母读取出来,遍历
			for d in self.__values[int(digits[index])]:
				# 处理index+1个数字,递归, cur要加当前的字符
				# 例如,如果输入了"23",index=0,那么递归时就时 (..., cur + a), (..., cur + b), (..., cur + c)
				self.__counter(digits, index + 1, cur + d)
		# 单独处理最后一个数字
		else:
			# 遍历最后一个数字对应的键盘中的字符,追加到结果中
			for d in self.__values[int(digits[index])]:
				self.__result.append(cur + d)


s = Solution()
print(s.letterCombinations('23'))
```