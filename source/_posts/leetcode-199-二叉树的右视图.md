title: leetcode --- 199. 二叉树的右视图
author: _Tao
tags: []
categories:
  - leetcode
date: 2021-04-05 23:22:00
---
199. 二叉树的右视图
author: _Tao
tags: []
categories:
  - leetcode
date: 2021-04-05 23:22:00
---
### 题目

[二叉树的右视图](https://leetcode-cn.com/problems/binary-tree-right-side-view)

给定一棵二叉树，想象自己站在它的右侧，按照从顶部到底部的顺序，返回从右侧所能看到的节点值。

示例
> 输入: [1,2,3,null,5,null,4] <br/> 
输出: [1, 3, 4] <br/>
解释:

```

   1            <---
 /   \
2     3         <---
 \     \
  5     4       <---
```


### 解法
```python
class TreeNode:
    def __init__(self, val=0, left=None, right=None):
        self.val = val
        self.left = left
        self.right = right


# 递归遍历
# class Solution:
#     def rightSideView(self, root: TreeNode) -> list:
#         if not root:
#             return []
#
#         rv = dict()
#         index = -1
#         self._traverse(root, rv, index)
#
#         keys = sorted(rv.keys())
#         result = [rv[k] for k in keys]
#         return result
#
#     def _traverse(self, node: TreeNode, rv: dict, index: int):
#         """
#         前序, 中序, 后序遍历皆可, 不影响结果
#         只需要保证右子树比左子树后遍历即可
#         """
#         index += 1
#         if node:
#             # 每一层保存更靠右的结果
#             rv[index] = node.val
#             self._traverse(node.left, rv, index + 1)
#             self._traverse(node.right, rv, index + 1)


# 层序遍历
class Solution:
    def rightSideView(self, root: TreeNode) -> list:
        if not root:
            return []
        rv = list()
        queue = [root]

        while queue:
            tmp = queue[:]
            queue.clear()
            for index, t in enumerate(tmp):
                if t.left:
                    queue.append(t.left)
                if t.right:
                    queue.append(t.right)
                # 保存每一层的最后一个数据
                if index == len(tmp) - 1:
                    rv.append(t.val)

        return rv


n1 = TreeNode(1)
n2 = TreeNode(2)
n3 = TreeNode(3)
n4 = TreeNode(4)
n5 = TreeNode(5)
n1.left = n2
n1.right = n3
n2.right = n5
# n3.right = n4

s = Solution()
print(s.rightSideView(n1))

```