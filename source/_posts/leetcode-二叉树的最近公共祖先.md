title: leetcode --- 二叉树的最近公共祖先
author: _Tao
tags: []
categories:
  - leetcode
date: 2020-05-23 17:50:00
---
[二叉树的最近公共祖先](https://leetcode-cn.com/problems/lowest-common-ancestor-of-a-binary-tree/)

给定一个二叉树, 找到该树中两个指定节点的最近公共祖先。

百度百科中最近公共祖先的定义为：“对于有根树 T 的两个结点 p、q，最近公共祖先表示为一个结点 x，满足 x 是 p、q 的祖先且 x 的深度尽可能大（一个节点也可以是它自己的祖先）。”

例如，给定如下二叉树:  root = [3,5,1,6,2,0,8,null,null,7,4]


![示例](https://qxinhai.oss-cn-shenzhen.aliyuncs.com/pic/32.png)


示例 1:

输入: root = [3,5,1,6,2,0,8,null,null,7,4], p = 5, q = 1

输出: 3

解释: 节点 5 和节点 1 的最近公共祖先是节点 3。<br/>

示例 2:

输入: root = [3,5,1,6,2,0,8,null,null,7,4], p = 5, q = 4

输出: 5

解释: 节点 5 和节点 4 的最近公共祖先是节点 5。因为根据定义最近公共祖先节点可以为节点本身。

说明:

所有节点的值都是唯一的。
p、q 为不同节点且均存在于给定的二叉树中。

<!-- more -->


解释:

1.如果二叉树中存在着该父节点，那么p一定在左子树上，q一定在右子树上

2.如果在某节点中的左子树中找到了p，右子树中找到了q，那么该节点就是最近的公共父节点

3.如果p，q都处于某节点的同一分支上，那么就需要沿着该分支继续遍历

```python
#!/usr/bin/env python
# -*- coding: utf-8 -*-
# @Author: _Tao
# @Time: 19-3-31 17:46


class TreeNode:
	def __init__(self, x):
		self.val = x
		self.left = None
		self.right = None

	def __str__(self):
		return 'val = ' + str(self.val)


class Solution:
	def lowestCommonAncestor(self, root: TreeNode, p: TreeNode, q: TreeNode) -> TreeNode:
		# 如果节点不存在，那么就是已经遍历到了最后
		# 如果是p，q，也就不再需要继续遍历
		if not root or root == p or root == q:
			return root

		# 遍历节点的左子树
		left_tree = self.lowestCommonAncestor(root.left, p, q)
		# 遍历节点的右子树
		right_tree = self.lowestCommonAncestor(root.right, p, q)
		
		# 如果左子树有数据，右子树也有数据，那么就说明是最终结果了
		if left_tree and right_tree:
			return root
		# 如果p，q都在右子树上
		elif not left_tree and right_tree:
			return right_tree
		# 如果p，q都在左子树上
		elif left_tree and not right_tree:
			return left_tree

		return None


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

t3.left = t5
t3.right = t1

t5.left = t6
t5.right = t2

t1.left = t0
t1.right = t8

t2.left = t7
t2.right = t4

print(so.lowestCommonAncestor(t3, t4, t8))

```