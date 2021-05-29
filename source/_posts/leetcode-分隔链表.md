title: Leetcode ---  分隔链表
author: _Tao
tags: []
categories:
  - Leetcode
date: 2020-05-23 19:03:00
---
### 题目
[分隔链表](https://leetcode-cn.com/problems/partition-list/submissions/)
<br/>

给定一个链表和一个特定值 x，对链表进行分隔，使得所有小于 x 的节点都在大于或等于 x 的节点之前。
你应当保留两个分区中每个节点的初始相对位置。

示例:
```
输入: head = 1->4->3->2->5->2, x = 3
输出: 1->2->2->4->3->5
```

<!-- more -->

### 题解

也是一个很简单的问题，熟悉一下golang的语法。
```go
package code

import "fmt"

type ListNode struct {
	Val  int
	Next *ListNode
}

func partition(head *ListNode, x int) *ListNode {

	var smallNode *ListNode
	var smallFirst *ListNode
	var bigNode *ListNode
	var bigFirst *ListNode

	if head == nil {
		return nil
	}

	for {
		if head == nil {
			break
		}
		if head.Val < x {
			if smallNode == nil {
				smallNode = head
				smallFirst = head
			} else {
				smallNode.Next = head
				smallNode = smallNode.Next
			}
		}
		if head.Val >= x {
			if bigNode == nil {
				bigNode = head
				bigFirst = head
			} else {
				bigNode.Next = head
				bigNode = bigNode.Next
			}
		}
		head = head.Next
	}

    // 最容易出错的地方
	if bigNode != nil {
		bigNode.Next = nil
	}
	if smallNode != nil {
		smallNode.Next = bigFirst
	}

	var node *ListNode
	if smallFirst != nil {
		node = smallFirst
	} else {
		node = bigFirst
	}

	return node

}

func Test86() {
	//	1->4->3->2->5->2, x = 3
	x := 0
	n1 := ListNode{Val: 1}
	n2 := ListNode{Val: 4}
	n3 := ListNode{Val: 3}
	n4 := ListNode{Val: 2}
	n5 := ListNode{Val: 5}
	n6 := ListNode{Val: 2}

	n1.Next = &n2
	n2.Next = &n3
	n3.Next = &n4
	n4.Next = &n5
	n5.Next = &n6

	node := partition(&n1, x)

	for {
		if node == nil {
			break
		}
		fmt.Println(node.Val)
		node = node.Next
	}

}

```