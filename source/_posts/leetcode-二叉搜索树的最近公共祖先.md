title: Leetcode --- 二叉搜索树的最近公共祖先
author: _Tao
tags: []
categories:
  - Leetcode
date: 2020-05-23 17:29:00
---
[二叉搜索树的最近公共祖先](https://leetcode-cn.com/problems/lowest-common-ancestor-of-a-binary-search-tree/)

给定一个二叉搜索树, 找到该树中两个指定节点的最近公共祖先。

百度百科中最近公共祖先的定义为：“对于有根树 T 的两个结点 p、q，最近公共祖先表示为一个结点 x，满足 x 是 p、q 的祖先且 x 的深度尽可能大（一个节点也可以是它自己的祖先）。”

例如，给定如下二叉搜索树:  root = [6,2,8,0,4,7,9,null,null,3,5]

![1.png](https://qxinhai.oss-cn-shenzhen.aliyuncs.com/pic/33.png)

示例 1:

输入: root = [6,2,8,0,4,7,9,null,null,3,5], p = 2, q = 8

输出: 6

解释: 节点 2 和节点 8 的最近公共祖先是 6。

示例 2:

输入: root = [6,2,8,0,4,7,9,null,null,3,5], p = 2, q = 4
输出: 2

解释: 节点 2 和节点 4 的最近公共祖先是 2, 因为根据定义最近公共祖先节点可以为节点本身。

说明:

所有节点的值都是唯一的。
p、q 为不同节点且均存在于给定的二叉搜索树中。

<!-- more -->

<br/>
这题相对于直接查找最近的父节点，还是容易很多的。搜索二叉树本身就已经排序好了的，只需要判断根节点比查找的节点大小即可。

```python
#!/usr/bin/env python
# -*- coding: utf-8 -*-
# @Author: _Tao
# @Time: 19-3-31 18:18


class TreeNode:
	def __init__(self, x):
		self.val = x
		self.left = None
		self.right = None

	def __str__(self):
		return 'val = ' + str(self.val)


class Solution:
	def lowestCommonAncestor(self, root: TreeNode, p: TreeNode, q: TreeNode) -> TreeNode:
		# 调整顺序，因为输入的节点中，有可能会出现p的值大于q的值的情况
		# 调整为，p值肯定小于q值
		if p.val > q.val:
			p, q = q, p
		# 按照搜索二叉树的概念，当节点的值大于p值切小于q值时，那么就是最近的父节点了
		# 注意等号的情况
		if p.val <= root.val <= q.val:
			return root
		# 如果root值大于p，q值，那么就遍历它的左子树
		elif root.val >= p.val and root.val >= q.val:
			return self.lowestCommonAncestor(root.left, p, q)
		# 如果root值小于p,q值，那么就遍历右子树
		elif root.val <= p.val and root.val <= q.val:
			return self.lowestCommonAncestor(root.right, p, q)


so = Solution()

t3 = TreeNode(3)
t5 = TreeNode(5)
t1 = TreeNode(1)
t6 = TreeNode(6)
t2 = TreeNode(2)
t0 = TreeNode(0)
t8 = TreeNode(8)
t7 = TreeNode(7)
t4 = TreeNode(4)
t9 = TreeNode(9)

t6.left = t2
t6.right = t8

t2.left = t0
t2.right = t4

t8.left = t7
t8.right = t9

t4.left = t3
t4.right = t5

print(so.lowestCommonAncestor(t6, t4, t3))

```