title: Leetcode --- 94. 二叉树的中序遍历
author: _Tao
tags: []
categories:
  - Leetcode
date: 2021-04-05 22:27:00
---
### 题目

[二叉树的中序遍历]()

给定一个二叉树的根节点 root ，返回它的 中序 遍历。

 
示例 1：
![](https://qxinhai.oss-cn-shenzhen.aliyuncs.com/hexo/20210405222801.png)

>输入：root = [1,null,2,3]<br/>
输出：[1,3,2]

### 题解
```python
# Definition for a binary tree node.
class TreeNode:
    def __init__(self, val=0, left=None, right=None):
        self.val = val
        self.left = left
        self.right = right


# 递归写法
# class Solution:
#     def inorderTraversal(self, root: TreeNode) -> list:
#         if not root:
#             return []
#
#         rv = list()
#         self._inorder(root, rv)
#
#         return rv
#
#     def _inorder(self, node, rv):
#         if node:
#             self._inorder(node.left, rv)
#             rv.append(node.val)
#             self._inorder(node.right, rv)

# 非递归
class Solution:
    def inorderTraversal(self, root: TreeNode) -> list:

        if not root:
            return []

        rv = list()
        stack = list()
        node = root

        while node or stack:
            # 如果node值存在, 则入栈
            if node:
                stack.append(node)
                # 左子树遍历
                node = node.left
            else:
                # 保存栈顶的值
                node = stack.pop(-1)
                rv.append(node.val)
                # 遍历右子树
                node = node.right

        return rv


n1 = TreeNode(1)
n2 = TreeNode(2)
n3 = TreeNode(3)
n1.right = n2
n2.left = n3

s = Solution()
print(s.inorderTraversal(n1))

```