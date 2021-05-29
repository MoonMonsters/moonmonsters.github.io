title: Leetcode --- 144. 二叉树的前序遍历
author: _Tao
tags: []
categories:
  - Leetcode
date: 2021-04-18 16:54:00
---
### 题目
[二叉树的前序遍历](https://leetcode-cn.com/problems/binary-tree-preorder-traversal/)

给你二叉树的根节点 root ，返回它节点值的 前序 遍历。

示例 1：
![](https://qxinhai.oss-cn-shenzhen.aliyuncs.com/hexo/20210418165503.png)

>输入：root = [1,null,2,3]<br/>
输出：[1,2,3]


### 解法
```python
# Definition for a binary tree node.
class TreeNode:
    def __init__(self, val=0, left=None, right=None):
        self.val = val
        self.left = left
        self.right = right


class Solution:
    def preorderTraversal(self, root: TreeNode) -> list:
        rv = list()
        self._pre2(root, rv)

        return rv

    def _pre(self, root, rv):
        """
        递归做法
        leetcode运行时间:40ms, 内存消耗:14.9mb
        """
        if root:
            rv.append(root.val)
            self._pre(root.left, rv)
            self._pre(root.right, rv)

    def _pre2(self, root, rv: list):
        """
        非递归做法
        运行时间:36ms, 内存消耗: 14.9mb
        """
        stack = []
        n = root
        while n or stack:
            while n:
                rv.append(n.val)
                stack.append(n)
                n = n.left
            if stack:
                t = stack.pop()
                n = t.right


t1 = TreeNode(1)
t2 = TreeNode(2)
t3 = TreeNode(3)

t1.right = t2
t2.left = t3

s = Solution()
print(s.preorderTraversal(t1))

```