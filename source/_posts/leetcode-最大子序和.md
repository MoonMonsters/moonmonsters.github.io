title: leetcode --- 最大子序和
author: _Tao
tags: []
categories:
  - leetcode
date: 2020-05-23 19:02:00
---
### 题目

[最大子序和](https://leetcode-cn.com/problems/maximum-subarray/)


给定一个整数数组 nums ，找到一个具有最大和的连续子数组（子数组最少包含一个元素），返回其最大和。

示例:
>
输入: [-2,1,-3,4,-1,2,1,-5,4],<br/>
输出: 6 <br/>
解释: 连续子数组 [4,-1,2,1] 的和最大，为 6。<br/>

<!-- more -->

### 题解

这道题很熟悉了，那时候大学时候在mooc上学习数据结构时就讲到了这道题，然后过了这么多年 ，每次刷题都能遇到，使用一个for循环就搞定。
```go
package code

import "fmt"

func maxSubArray(nums []int) int {
	var result = nums[0]
	var tmp = 0
	for _, v := range nums {
		tmp += v
		if tmp > result {
			result = tmp
		}
		if tmp < 0 {
			tmp = 0
		}
	}

	return result
}

func Test53() {
	var nums = []int{-2, 1, -3, 4, -1, 2, 1, -5, 4}
	fmt.Println(maxSubArray(nums))
}

```