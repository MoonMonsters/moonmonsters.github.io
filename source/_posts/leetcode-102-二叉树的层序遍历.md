title: Leetcode --- 102. 二叉树的层序遍历
author: _Tao
tags: []
categories:
  - Leetcode
date: 2021-03-27 14:32:00
---
### 题目

[二叉树的层序遍历](https://leetcode-cn.com/problems/binary-tree-level-order-traversal)

给你一个二叉树，请你返回其按 层序遍历 得到的节点值。 （即逐层地，从左到右访问所有节点）。

 
示例：
> 二叉树：[3,9,20,null,null,15,7],

``` 
	3
   / \
  9  20
    /  \
   15   7
```
返回其层序遍历结果：
```
[
  [3],
  [9,20],
  [15,7]
]
```


### 解法
```python
class TreeNode:
    def __init__(self, val=0, left=None, right=None):
        self.val = val
        self.left = left
        self.right = right


class Solution:
    def levelOrder(self, root: TreeNode) -> list:
        if not root:
            return []

        # 判断层序是否完成
        queue = [[root]]
        # 返回结果
        result = list()
        while queue:
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
print(s.levelOrder(n3))

```

