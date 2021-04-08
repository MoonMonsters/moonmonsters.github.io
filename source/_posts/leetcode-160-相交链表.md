title: leetcode --- 160. 相交链表
author: _Tao
tags: []
categories:
  - leetcode
date: 2021-03-27 14:49:00
---
### 题目

[相交链表](https://leetcode-cn.com/problems/intersection-of-two-linked-lists)

编写一个程序，找到两个单链表相交的起始节点。

如下面的两个链表：
![](https://qxinhai.oss-cn-shenzhen.aliyuncs.com/hexo/20210327144950.png)
在节点 c1 开始相交。

示例
![](https://qxinhai.oss-cn-shenzhen.aliyuncs.com/hexo/20210327145018.png)
>输入：intersectVal = 8, listA = [4,1,8,4,5], listB = [5,0,1,8,4,5], skipA = 2, skipB = 3 <br/>
输出：Reference of the node with value = 8 <br/>
输入解释：相交节点的值为 8 （注意，如果两个链表相交则不能为 0）。从各自的表头开始算起，链表 A 为 [4,1,8,4,5]，链表 B 为 [5,0,1,8,4,5]。在 A 中，相交节点前有 2 个节点；在 B 中，相交节点前有 3 个节点。<br/>

### 解法
```python
# Definition for singly-linked list.
class ListNode:
    def __init__(self, x):
        self.val = x
        self.next = None


class Solution:
    def getIntersectionNode(self, headA: ListNode, headB: ListNode) -> ListNode:
        la = 0
        na = headA
        while na:
            la += 1
            na = na.next

        lb = 0
        nb = headB
        while nb:
            lb += 1
            nb = nb.next

        length = abs(la - lb)
        if la > lb:
            for i in range(length):
                headA = headA.next
        else:
            for i in range(length):
                headB = headB.next

        while headB and headA:
            if headB == headA:
                return headA
            headA = headA.next
            headB = headB.next


s = Solution()
listA = [4, 1, 8, 4, 5]
listB = [5, 0, 1, 8, 4, 5]

tailA = None
headA = None
for a in listA[::-1]:
    n = ListNode(a)
    n.next = tailA
    tailA = n
    headA = n

tailB = None
headB = None
for a in listB[::-1]:
    n = ListNode(a)
    n.next = tailB
    tailB = n
    headB = n

# while headA:
#     print(headA.val)
#     headA = headA.next
# print('-' * 80)
#
# while headB:
#     print(headB.val)
#     headB = headB.next
# print('=' * 80)

s = Solution()
node = s.getIntersectionNode(headA, headB)
print(node.val)

```


