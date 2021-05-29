title: Leetcode --- 21. 合并两个有序链表
author: _Tao
tags: []
categories:
  - Leetcode
date: 2021-03-27 12:58:00
---
### 题目
[合并两个有序链表]()

将两个升序链表合并为一个新的 升序 链表并返回。新链表是通过拼接给定的两个链表的所有节点组成的。 

示例 1：
> ![](https://qxinhai.oss-cn-shenzhen.aliyuncs.com/hexo/20210327125926.png)
输入：l1 = [1,2,4], l2 = [1,3,4] <br/>
输出：[1,1,2,3,4,4]

示例 2：
> 输入：l1 = [], l2 = []
输出：[]

示例 3：
> 输入：l1 = [], l2 = [0]
输出：[0]


### 解法
```python
# Definition for singly-linked list.
class ListNode:
    def __init__(self, val=0, next=None):
        self.val = val
        self.next = next


class Solution:
    def mergeTwoLists(self, l1: ListNode, l2: ListNode) -> ListNode:

        if not l1 and not l2:
            return None
        elif l1 and not l2:
            return l1
        elif not l1 and l2:
            return l2

        cur = None
        head = None
        while l1 and l2:
            if l1.val <= l2.val:
                if not head:
                    head = l1
                    cur = head
                else:
                    cur.next = l1
                    cur = cur.next
                l1 = l1.next
            else:
                if not head:
                    head = l2
                    cur = head
                else:
                    cur.next = l2
                    cur = cur.next
                l2 = l2.next

        if l1:
            cur.next = l1
        if l2:
            cur.next = l2

        return head


l1 = [1, 2, 4]
l2 = [1, 3, 4]

la1 = ListNode(1)
la2 = ListNode(2)
la3 = ListNode(4)
la1.next = la2
la2.next = la3

lb1 = ListNode(1)
lb2 = ListNode(3)
lb3 = ListNode(4)
lb1.next = lb2
lb2.next = lb3

s = Solution()
node = s.mergeTwoLists(la1, lb1)
while node:
    print(node.val)
    node = node.next

```