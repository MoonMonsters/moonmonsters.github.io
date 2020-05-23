title: leetcode --- 合并K个有序链表
author: _Tao
tags: []
categories:
  - leetcode
date: 2020-05-23 17:51:00
---
[合并K个有序链表](https://leetcode-cn.com/problems/merge-k-sorted-lists/)

合并 k 个排序链表，返回合并后的排序链表。请分析和描述算法的复杂度。

示例:

输入:
[
  1->4->5,
  1->3->4,
  2->6
]
输出: 1->1->2->3->4->4->5->6

<!-- more -->

<br/>
<br/>
方法跟合并两个链表差不多，不过要采用分而治之的方法.....
之前采用reduce写超时了，也肯定得超时，是得多蠢...

```python
#!/usr/bin/env python
# -*- coding: utf-8 -*-
# @Author: _Tao
# @Time: 19-3-30 17:10


class ListNode:
	def __init__(self, x):
		self.val = x
		self.next = None


class Solution:

	def mergeKLists(self, lists) -> ListNode:
		# 几种特殊情况
		if not lists:
			return lists
		elif len(lists) == 1:
			return lists[0]
		elif len(lists) == 2:
			return self.mergeTwoLists(lists[0], lists[1])

		# 求得中间的值
		if len(lists) % 2 == 0:
			mid = len(lists) // 2
		else:
			mid = (len(lists) + 1) // 2
		mid -= 1

		# 采用分而治之的方法求
		# 一直二分下去，最后合并
		return self.mergeTwoLists(self.mergeKLists(lists[:mid]), self.mergeKLists(lists[mid:]))

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

x1 = ListNode(1)
x2 = ListNode(10)
x1.next = x2

so = Solution()
node = so.mergeKLists([m1, n1, x1])

while node:
	print(node.val)
	node = node.next
```