title: Leetcode --- 合并两个有序链表
author: _Tao
tags: []
categories:
  - Leetcode
date: 2020-05-23 18:54:00
---
[合并两个有序链表](https://leetcode-cn.com/problems/merge-two-sorted-lists/)

将两个有序链表合并为一个新的有序链表并返回。新链表是通过拼接给定的两个链表的所有节点组成的。 

示例：

输入：1->2->4, 1->3->4
输出：1->1->2->3->4->4
<br/>

<!-- more -->

题目很简单，就是普通链表做法。
```python
#!/usr/bin/env python
# -*- coding: utf-8 -*-
# @Author: _Tao
# @Time: 19-3-30 15:50


class ListNode:
	def __init__(self, x):
		self.val = x
		self.next = None


class Solution:
	def mergeTwoLists(self, l1: ListNode, l2: ListNode) -> ListNode:
		cur_node = None
		head = None

		# 判断是否有数据为空的情况
		if not l1:
			return l2

		if not l2:
			return l1

		# 遍历判断大小
		while l1 and l2:
			if l1.val < l2.val:
				val = l1.val
				# 移动节点
				l1 = l1.next
			else:
				val = l2.val
				l2 = l2.next
			# 创建新节点
			node = ListNode(val)
			# 设置头节点
			if not head:
				cur_node = node
				head = cur_node
			else:
				cur_node.next = node
				cur_node = node

		# 将没有遍历完的数据，追加到节点之后
		while l1:
			node = ListNode(l1.val)
			l1 = l1.next
			cur_node.next = node
			cur_node = node

		while l2:
			node = ListNode(l2.val)
			l2 = l2.next
			cur_node.next = node
			cur_node = node

		return head


m1 = ListNode(1)
m2 = ListNode(2)
m3 = ListNode(4)

m1.next = m2
m2.next = m3

n1 = ListNode(1)
n2 = ListNode(3)
n3 = ListNode(4)

n1.next = n2
n2.next = n3

so = Solution()
print(so.mergeTwoLists(m1, n1).val)

```