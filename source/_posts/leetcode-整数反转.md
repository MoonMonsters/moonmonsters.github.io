title: Leetcode --- 整数反转
author: _Tao
tags: []
categories:
  - Leetcode
date: 2020-05-23 17:55:00
---
[整数反转](https://leetcode-cn.com/problems/reverse-integer/)

给出一个 32 位的有符号整数，你需要将这个整数中每位上的数字进行反转。

示例 1:
输入: 123
输出: 321

 示例 2:
输入: -123
输出: -321

示例 3:
输入: 120
输出: 21

注意:
假设我们的环境只能存储得下 32 位的有符号整数，则其数值范围为 [−2<sup>31</sup>,  2<sup>31</sup> − 1]。请根据这个假设，如果反转后整数溢出那么就返回 0。

<!-- more -->

没什么难度,硬刚就成.

```python
#!/usr/bin/env python
# -*- coding: utf-8 -*-
# @Author: _Tao
# @Time: 19-5-8 23:43

class Solution:
	def reverse(self, x: int) -> int:
		if x == 0:
			y = 0
		elif x > 0:
			y = int(''.join(list(reversed(str(x)))))
		else:
			y = int(str('-') + ''.join(list(reversed(str(-x)))))

		if -2 ** 31 < y < 2 ** 31 - 1:
			return y
		return 0
```