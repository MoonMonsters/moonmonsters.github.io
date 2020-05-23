title: leetcode --- 两数相加
author: _Tao
tags: []
categories:
  - leetcode
date: 2020-05-23 18:54:00
---
[两数相加](https://leetcode-cn.com/problems/add-two-numbers/)

给出两个 非空 的链表用来表示两个非负的整数。其中，它们各自的位数是按照 逆序 的方式存储的，并且它们的每个节点只能存储 一位 数字。
如果，我们将这两个数相加起来，则会返回一个新的链表来表示它们的和。
您可以假设除了数字 0 之外，这两个数都不会以 0 开头。

示例：
输入：(2 -> 4 -> 3) + (5 -> 6 -> 4)
输出：7 -> 0 -> 8
原因：342 + 465 = 807

<!-- more -->


一脸懵逼，用最正常的解法做出来的，虽然时间和空间复杂度都有点高，但还是过了。
看评论说，O(2n)的时间复杂度都过不去，那我这写法怎么过的？

```python
# !/usr/bin/env python
# -*- coding:utf-8 -*-
# __author__ = '_Tao'

class ListNode:
    def __init__(self, x):
        self.val = x
        self.next = None


class Solution:
    def addTwoNumbers(self, l1: ListNode, l2: ListNode) -> ListNode:
        num1 = self.get_num(l1)
        num2 = self.get_num(l2)

        return self.split_num(num1 + num2)

    def get_num(self, node):
        num = 0
        index = 0
        while node:
            num += node.val * pow(10, index)
            index += 1
            node = node.next

        return num

    def split_num(self, num):
        s = str(num)
        cur = None
        while len(s) >= 1:
            last = s[0]
            s = s[1:]
            x = ListNode(int(last))
            x.next = cur
            cur = x

        return cur


if __name__ == '__main__':
    s = Solution()
    n1 = ListNode(2)
    n2 = ListNode(4)
    n3 = ListNode(3)

    n1.next = n2
    n2.next = n3

    n4 = ListNode(5)
    n5 = ListNode(6)
    n6 = ListNode(4)

    n4.next = n5
    n5.next = n6

    x = s.addTwoNumbers(n1, n4)
    print(s.get_num(x))

```