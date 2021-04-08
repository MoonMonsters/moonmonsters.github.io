title: leetcode --- 旋转链表
author: _Tao
tags: []
categories: []
date: 2021-03-07 16:18:00
---
### 题目
[旋转链表](https://leetcode-cn.com/problems/rotate-list/)

给定一个链表，旋转链表，将链表每个节点向右移动 k 个位置，其中 k 是非负数。

示例1
>输入: 1->2->3->4->5->NULL, k = 2 <br/>
输出: 4->5->1->2->3->NULL <br/>	
解释:	<br/>
向右旋转 1 步: 5->1->2->3->4->NULL <br/>
向右旋转 2 步: 4->5->1->2->3->NULL	<br/>

示例2
>输入: 0->1->2->NULL, k = 4	<br/>
输出: 2->0->1->NULL	<br/>
解释:	<br/>
向右旋转 1 步: 2->0->1->NULL	<br/>
向右旋转 2 步: 1->2->0->NULL	<br/>
向右旋转 3 步: 0->1->2->NULL	<br/>
向右旋转 4 步: 2->0->1->NULL	<br/>


<!-- more -->

### 解法
```python
class ListNode:
    def __init__(self, val=0, next=None):
        self.val = val
        self.next = next


class Solution:
    def rotateRight(self, head: ListNode, k: int) -> ListNode:
        if not head or not head.next or k == 0:
            return head

        length = 0
        tail = head

        while tail:
            length += 1
            if tail.next:
                tail = tail.next
            else:
                break

        target = k % length
        if target == 0:
            return head

        index = 0
        # 构建循环链表
        tail.next = head

        while index < length - target:
            tail = tail.next
            index += 1

        # 新的尾节点的下一个节点, 作为头结点
        new_head = tail.next
        tail.next = None

        return new_head


def links(nums):
    head = None
    cur = None
    for n in nums:
        if head is None:
            head = ListNode(n)
            cur = head
        else:
            node = ListNode(n)
            cur.next = node
            cur = node

    return head


def traverse(head):
    cur = head
    while cur:
        print(cur.val, end=" ")
        cur = cur.next
    print("\n" + "-" * 80)


if __name__ == '__main__':
    nums = [1, 2, 3, 4, 5]
    head = links(nums)
    traverse(head)
    s = Solution()
    new_head = s.rotateRight(head, 2)
    traverse(new_head)

```