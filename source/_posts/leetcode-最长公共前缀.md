title: leetcode --- 最长公共前缀
author: _Tao
tags: []
categories:
  - leetcode
date: 2020-05-23 17:58:00
---
编写一个函数来查找字符串数组中的最长公共前缀。

如果不存在公共前缀，返回空字符串 `""`。

**示例 1:**

```
输入: ["flower","flow","flight"]
输出: "fl"
```

**示例 2:**

```
输入: ["dog","racecar","car"]
输出: ""
解释: 输入不存在公共前缀。
```



<!-- more -->

常规做法:

从第一个字符开始，判断到最后一个字符，如果某次不相等，或者全部匹配完成，就返回保存公共前缀的临时变量，时间复杂度时O(n*m)

```python
class Solution(object):
	def longestCommonPrefix(self, strs):
		"""
		:type strs: List[str]
		:rtype: str
		"""
		if not strs:
			return ''
		if len(strs) == 1:
			return strs[0]

		index = 0
		cur = ''
		while True:
			try:
				istr = strs[0][index]
				for s in strs[1:]:
					if s[index] != istr:
						return cur
				else:
					index += 1
					cur = ''.join([cur, istr])
			except:
				return cur
```



看了下评论区的解法，给跪了...

给字符串排序，只需要判断“最大字符串”和“最小字符串”的公共前缀即可。

```python
class Solution(object):
	def longestCommonPrefix(self, strs):
		"""
		:type strs: List[str]
		:rtype: str
		"""
		if not strs: return ""
		maxstr = max(strs)
		minstr = min(strs)

		for index, value in enumerate(minstr):
			if value != maxstr[index]:
				return maxstr[:index]

		return minstr
```