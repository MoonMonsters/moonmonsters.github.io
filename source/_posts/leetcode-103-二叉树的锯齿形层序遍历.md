title: leetcode --- 103. 二叉树的锯齿形层序遍历
author: _Tao
tags: []
categories:
  - leetcode
date: 2021-03-27 14:35:00
---
### 题目

[二叉树的锯齿形层序遍历](https://leetcode-cn.com/problems/binary-tree-zigzag-level-order-traversal)

给定一个二叉树，返回其节点值的锯齿形层序遍历。（即先从左往右，再从右往左进行下一层遍历，以此类推，层与层之间交替进行）。

例如：
给定二叉树 [3,9,20,null,null,15,7],
```
    3
   / \
  9  20
    /  \
   15   7
```
返回锯齿形层序遍历如下：
```
[
  [3],
  [20,9],
  [15,7]
]
```

### 解法
```python
# Definition for a binary tree node.
class TreeNode:
    def __init__(self, val=0, left=None, right=None):
        self.val = val
        self.left = left
        self.right = right


class Solution:
    def zigzagLevelOrder(self, root: TreeNode) -> list:
        if not root:
            return []

        # 判断层序是否完成
        queue = [[root]]
        # 返回结果
        result = list()
        # 用来判断奇偶性
        index = 0
        while queue:
            index += 1
            # 临时存储每一层的节点的值
            tmp_val = list()
            # 临时存储每一层节点, 从左往右
            tmp_node = list()
            nodes = queue.pop(0)
            # 从左往右遍历每一个节点, 保存节点以及节点数据
            for node in nodes:
                tmp_val.append(node.val)
                if node.left:
                    tmp_node.append(node.left)
                if node.right:
                    tmp_node.append(node.right)
            # 如果是偶数层, 那么逆序输出
            if index % 2 == 0:
                tmp_val = tmp_val[::-1]
            # 不能存储空列表, 这个是循环退出条件
            if tmp_node:
                queue.append(tmp_node)
            result.append(tmp_val)

        return result


n3 = TreeNode(3)
n9 = TreeNode(9)
n20 = TreeNode(20)
n15 = TreeNode(15)
n7 = TreeNode(7)

n3.left = n9
n3.right = n20
n20.left = n15
n20.right = n7

s = Solution()
print(s.zigzagLevelOrder(n3))

```