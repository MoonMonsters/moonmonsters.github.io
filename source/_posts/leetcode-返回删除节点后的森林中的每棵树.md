title: leetcode --- 返回删除节点后的森林中的每棵树
author: _Tao
tags: []
categories:
  - leetcode
date: 2020-05-23 18:59:00
---
### 题目

这个题目是leetcode上随机产生的一个面试题，所以没有链接了。



给出二叉树的根节点 `root`，树上每个节点都有一个不同的值。

如果节点值在 `to_delete` 中出现，我们就把该节点从树上删去，最后得到一个森林（一些不相交的树构成的集合）。

返回森林中的每棵树。你可以按任意顺序组织答案。

示例:

![](https://qxinhai.oss-cn-shenzhen.aliyuncs.com/blog/20190803/screen-shot-2019-07-01-at-53836-pm.png)

```text
输入：root = [1,2,3,4,5,6,7], to_delete = [3,5]
输出：[[1,2,null,4],[6],[7]]
```

提示:

- 树中的节点数最大为 `1000`。
- 每个节点都有一个介于 `1` 到 `1000` 之间的值，且各不相同。
- `to_delete.length <= 1000`
- `to_delete` 包含一些从 `1` 到 `1000`、各不相同的值。

<!-- more -->

### 代码

主要需要处理删除的节点以及子节点的问题。

要想通过测试用例，则需要把要删除的节点的父节点的left \ right赋值为None才行。

主要分析见代码注释即可。

特别需要注意的是，结果返回的是一个列表，列表中存放着森林中每棵二叉树的根节点，不是直接返回最终结果....

```python
class TreeNode:
    def __init__(self, x):
        self.val = x
        self.left = None
        self.right = None

    def __str__(self):
        return str(self.val)

    __repr__ = __str__


class Solution:
    def delNodes(self, root: TreeNode, to_delete: list) -> list:
        # 过程，凡是树的根节点的都放在此列表中
        # 就是根节点和要删除的节点
        process = list()
        # 结果
        result = list()

        process.append(root)
        while len(process) > 0:
            # 抛出一个节点
            node = process.pop(0)
            # 遍历删除
            # 如果这个节点不是删除的节点，并且存在，就添加进结果中
            # 这个判断条件不能不添加，否则会通不过这组数据
            # root = [1, 2, None, 4, 3]
            # to_delete = [2, 3]
            if not self.traverse(node, process, to_delete) and node:
                result.append(node)

        return result

    def traverse(self, node, process, to_delete):
        if node:
            # 如果当前节点的值不是要删除的节点，那么就继续往左\右遍历
            if node.val not in to_delete:
                # 为了能通过测试提交，如果返回的是True，就表示左\右子节点删除了
                # 那么就需要把父节点的左\右指针置空
                if self.traverse(node.left, process, to_delete):
                    node.left = None
                if self.traverse(node.right, process, to_delete):
                    node.right = None
            else:
                # 需要删除本身
                # 把左右两个子节点加入列表中，需要往下遍历
                process.append(node.left)
                process.append(node.right)

                return True


# 测试代码
root = [1, 2, None, 4, 3]
to_delete = [2, 3]


def create():
    # 层序构造二叉树
    # 构造的是完全二叉树，所以此处不会存在bug
    # 如果是普通二叉树，不能这么写了
    nodes = list()
    for index, num in enumerate(root):
        node = TreeNode(num)
        # 根节点
        if index == 0:
            pass
        # 左孩子节点
        elif index % 2 == 1:
            parent = int((index - 1) / 2)
            parent_node = nodes[parent]
            parent_node.left = node
        # 右孩子节点
        elif index % 2 == 0:
            parent = int((index - 2) / 2)
            parent_node = nodes[parent]
            parent_node.right = node

        nodes.append(node)

    return nodes

print(create())
head = create()[0]
s = Solution()
print(s.delNodes(head, to_delete))
```