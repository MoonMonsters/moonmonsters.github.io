title: Leetcode --- 爬楼梯
author: _Tao
tags: []
categories:
  - Leetcode
date: 2020-05-23 19:02:00
---
### 题目
[爬楼梯](https://leetcode-cn.com/problems/climbing-stairs/)
<br/>
假设你正在爬楼梯。需要 n 阶你才能到达楼顶。
每次你可以爬 1 或 2 个台阶。你有多少种不同的方法可以爬到楼顶呢？
注意：给定 n 是一个正整数。

示例 1：
```
输入： 2 
输出： 2
解释： 有两种方法可以爬到楼顶。
1.  1 阶 + 1 阶
2.  2 阶
```

示例 2：
```
输入： 3
输出： 3
解释： 有三种方法可以爬到楼顶。
1.  1 阶 + 1 阶 + 1 阶
2.  1 阶 + 2 阶
3.  2 阶 + 1 阶
```

<!-- more -->

### 题解

很常见的一道题目了，用常规的递归解决就可以了。
没想到的是，加上map做缓存后，耗时竟然是0ms，
```
执行用时 :0 ms, 在所有 Go 提交中击败了100.00% 的用户
内存消耗 :1.9 MB, 在所有 Go 提交中击败了76.09%的用户
```

```golang
package code

import "fmt"

// 缓存每一个n的值
var cache = make(map[int]int)

func climbStairs(n int) int {
	// 递归的结束条件
	if n <= 2 {
		return n
	}
	// 获取缓存
	if _, ok := cache[n]; ok {
		return cache[n]
	}
	m := climbStairs(n-1) + climbStairs(n-2)
	cache[n] = m

	return m
}

func Test70() {
	fmt.Println(climbStairs(2))
}
```