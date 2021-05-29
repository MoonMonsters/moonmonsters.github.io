title: Leetcode --- 236. 二叉树的最近公共祖先
author: _Tao
tags: []
categories:
  - Leetcode
date: 2021-03-27 14:58:00
---
### 题目

[二叉树的最近公共祖先](https://leetcode-cn.com/problems/lowest-common-ancestor-of-a-binary-tree)

给定一个二叉树, 找到该树中两个指定节点的最近公共祖先。

百度百科中最近公共祖先的定义为：“对于有根树 T 的两个节点 p、q，最近公共祖先表示为一个节点 x，满足 x 是 p、q 的祖先且 x 的深度尽可能大（一个节点也可以是它自己的祖先）。”

示例
![](https://qxinhai.oss-cn-shenzhen.aliyuncs.com/hexo/20210327145858.png)
>输入：root = [3,5,1,6,2,0,8,null,null,7,4], p = 5, q = 1 <br/>
输出：3 <br/>
解释：节点 5 和节点 1 的最近公共祖先是节点 3 。<br/>

### 解法
```python
# Definition for a binary tree node.
class TreeNode:
    def __init__(self, x):
        self.val = x
        self.left = None
        self.right = None


class Solution:
    def lowestCommonAncestor(self, root: TreeNode, p: TreeNode, q: TreeNode) -> TreeNode:

        node_dict = self._traverse(root)
        # 需要加入自身节点
        qSet = {q}
        pSet = {p}

        while True:
            # 若有重合节点, 则重合的节点就是返回值
            result = qSet & pSet
            if len(result) > 0:
                return result.pop()

            # 获取父节点
            qParent = node_dict.get(q.val)
            pParent = node_dict.get(p.val)

            # 将父节点加入到搜索路径中
            if qParent:
                qSet.add(qParent)
                q = qParent

            if pParent:
                pSet.add(pParent)
                p = pParent

    def _traverse(self, root: TreeNode):
        """
        构建字典, {子节点val: 父节点}的形式, 因为题目中提到val不会重复
        """
        q: list = [root]
        node_dict = {}
        while q:
            node = q.pop(0)
            left = node.left
            right = node.right
            if left:
                q.append(left)
                node_dict[left.val] = node
            if right:
                q.append(right)
                node_dict[right.val] = node

        return node_dict


n3 = TreeNode(3)
n5 = TreeNode(5)
n1 = TreeNode(1)
n6 = TreeNode(6)
n0 = TreeNode(0)
n7 = TreeNode(7)
n2 = TreeNode(2)
n8 = TreeNode(8)
n4 = TreeNode(4)

n3.left = n5
n3.right = n1
n5.left = n6
n5.right = n2
n1.left = n0
n1.right = n8
n2.left = n7
n2.right = n4

s = Solution()
node = s.lowestCommonAncestor(n3, n5, n4)
print(node.val)

```