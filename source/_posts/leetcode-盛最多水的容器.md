title: leetcode --- 盛最多水的容器
author: _Tao
tags: []
categories:
  - leetcode
date: 2020-05-23 17:56:00
---
[盛最多水的容器](https://leetcode-cn.com/problems/container-with-most-water/)

给定 n 个非负整数 a1，a2，...，an，每个数代表坐标中的一个点 (i, ai) 。在坐标内画 n 条垂直线，垂直线 i 的两个端点分别为 (i, ai) 和 (i, 0)。找出其中的两条线，使得它们与 x 轴共同构成的容器可以容纳最多的水。

说明：你不能倾斜容器，且 n 的值至少为 2。
![](https://qxinhai.oss-cn-shenzhen.aliyuncs.com/pic/question_11.jpg)
图中垂直线代表输入数组 (1,8,6,2,5,4,8,3,7)。在此情况下，容器能够容纳水（表示为蓝色部分）的最大值为 49。

示例:

输入: (1,8,6,2,5,4,8,3,7)
输出: 49

<!-- more -->

其实就是求最大面积.
一个索引指向最左边,一直指向最右边,然后判断两个索引处的值的大小,哪个小,就将对应的索引往中心靠,直到左边的大于等于右边即可,取得最大值.

```python
#!/usr/bin/env python
# -*- coding: utf-8 -*-
# @Author: _Tao
# @Time: 19-5-8 23:58


class Solution:
	def maxArea(self, height) -> int:
		start = 0
		end = len(height) - 1
		maxnum = 0
		while end > start:
			startnum = height[start]
			endnum = height[end]
			if min(startnum, endnum) * (end - start) > maxnum:
				maxnum = min(startnum, endnum) * (end - start)
			if startnum < endnum:
				start += 1
			else:
				end -= 1
		return maxnum
```