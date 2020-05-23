title: leetcode ---  颜色分类
author: _Tao
tags: []
categories:
  - leetcode
date: 2020-05-23 19:03:00
---
### 题目
[颜色分类](https://leetcode-cn.com/problems/sort-colors/)
<br/>
给定一个包含红色、白色和蓝色，一共 n 个元素的数组，原地对它们进行排序，使得相同颜色的元素相邻，并按照红色、白色、蓝色顺序排列。

此题中，我们使用整数 0、 1 和 2 分别表示红色、白色和蓝色。

注意:
不能使用代码库中的排序函数来解决这道题。

示例:
```
输入: [2,0,2,1,1,0]
输出: [0,0,1,1,2,2]
```

<!-- more -->

### 题解

题目虽然是中等难度，但还是很简单吧，就是桶排序的意思。
```golang
func sortColors(nums []int) {
	var colors = [3]int{}
	for i := 0; i < len(nums); i++ {
		colors[nums[i]] += 1
	}
	index := 0
	for i := 0; i < len(colors); i++ {
		for j := 0; j < colors[i]; j++ {
			nums[index] = i
			index++
		}
	}
}
```