title: leetcode---无重复字符的最长子串
author: _Tao
tags: []
categories:
  - leetcode
date: 2020-05-23 17:52:00
---
[无重复字符的最长子串](https://leetcode-cn.com/problems/longest-substring-without-repeating-characters/)
<br/>
给定一个字符串，请你找出其中不含有重复字符的 最长子串 的长度。

输入: "abcabcbb"
输出: 3 
解释: 因为无重复字符的最长子串是 "abc"，所以其长度为 3。

<!-- more -->


跟以前做的从一组数字中取出最大子序列之和一样，虽然一遍通过，但复杂度O(n)貌似还是挺高的。

```python
#!/usr	/bin/env python
# -*- coding: utf-8 -*-
# @Author: _Tao
# @Time: 19-3-27 23:07


class Solution:
	def lengthOfLongestSubstring(self, s: str) -> int:
		non_repeat = ''
		max_length = 0
		for ch in s:
			# 如果没有重复的，就追加到字符串
			if ch not in non_repeat:
				non_repeat += ch
			else:
				# 否则，截取重复的字符之后的字符串
				index = non_repeat.index(ch)
				non_repeat = non_repeat[index + 1:] + ch
			# 获取最大长度
			max_length = max(max_length, len(non_repeat))
		return max_length


sol = Solution()
xxx = 'bbbbb'
print(sol.lengthOfLongestSubstring(xxx))

```