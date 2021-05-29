title: Leetcode --- 2. 两数相加
author: _Tao
tags: []
categories:
  - Leetcode
date: 2021-04-18 14:06:00
---
### 题目
[两数相加](https://leetcode-cn.com/problems/add-two-numbers/)

给你两个 非空 的链表，表示两个非负的整数。它们每位数字都是按照 逆序 的方式存储的，并且每个节点只能存储 `一位` 数字。
请你将两个数相加，并以相同形式返回一个表示和的链表。
你可以假设除了数字 0 之外，这两个数都不会以 0 开头。

![](https://qxinhai.oss-cn-shenzhen.aliyuncs.com/hexo/20210418140849.png)
>输入：l1 = [2,4,3], l2 = [5,6,4] <br/>
输出：[7,0,8]	<br/>
解释：342 + 465 = 807.

### 解法
```python
# Definition for singly-linked list.
class ListNode:
    def __init__(self, val=0, next=None):
        self.val = val
        self.next = next


class Solution:
    def addTwoNumbers(self, l1: ListNode, l2: ListNode) -> ListNode:

        if not l1:
            return l2
        if not l2:
            return l1

        head = None
        tail = None
        tmp = 0

        while l1 and l2:
            val = l1.val + l2.val + tmp
            tmp = val // 10
            l1 = l1.next
            l2 = l2.next

            n = ListNode(val % 10)
            if head is None:
                head = n
                tail = n
            else:
                tail.next = n
                tail = n

        m = l1 if l1 else l2
        while m:
            val = m.val + tmp
            n = ListNode(val % 10)
            tail.next = n
            tail = n
            tmp = val // 10
            m = m.next

        if tmp > 0:
            n = ListNode(tmp)
            tail.next = n

        return head


# l1_2 = ListNode(2)
# l1_4 = ListNode(4)
# l1_3 = ListNode(3)
# l1_2.next = l1_4
# l1_4.next = l1_3
#
# l2_5 = ListNode(5)
# l2_6 = ListNode(6)
# l2_4 = ListNode(4)
# l2_5.next = l2_6
# l2_6.next = l2_4

l1_1 = ListNode(9)
l1_2 = ListNode(9)
l1_3 = ListNode(1)
l1_1.next = l1_2
l1_2.next = l1_3

l2_1 = ListNode(1)

s = Solution()
cur = s.addTwoNumbers(l1_1, l2_1)
while cur:
    print("cur: %s" % cur.val)
    cur = cur.next

```