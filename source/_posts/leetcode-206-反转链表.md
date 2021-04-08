title: leetcode --- 206. 反转链表
author: _Tao
tags: []
categories:
  - leetcode
date: 2021-03-27 14:54:00
---
### 题目

[反转链表](https://leetcode-cn.com/problems/reverse-linked-list/)

反转一个单链表。

示例:
> 输入: 1->2->3->4->5->NULL <br/>
输出: 5->4->3->2->1->NULL



### 解法
```python
# Definition for singly-linked list.
class ListNode:
    def __init__(self, val=0, next=None):
        self.val = val
        self.next = next


class Solution:
    def reverseList(self, head: ListNode) -> ListNode:
        cur = None
        while head:
            cur = ListNode(head.val, cur)
            head = head.next

        return cur


def traverse(head):
    while head:
        print(head.val)
        head = head.next


n1 = ListNode(1)
n2 = ListNode(2)
n3 = ListNode(3)
n4 = ListNode(4)
n5 = ListNode(5)

n1.next = n2
n2.next = n3
n3.next = n4
n4.next = n5
traverse(n1)
print('-' * 80)

s = Solution()
n = s.reverseList(n1)
traverse(n)

```