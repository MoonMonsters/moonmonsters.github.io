title: Leetcode --- 141. 环形链表
author: _Tao
tags: []
categories:
  - Leetcode
date: 2021-03-27 14:42:00
---
### 题目

[环形链表](https://leetcode-cn.com/problems/linked-list-cycle)

给定一个链表，判断链表中是否有环。

如果链表中有某个节点，可以通过连续跟踪 next 指针再次到达，则链表中存在环。 为了表示给定链表中的环，我们使用整数 pos 来表示链表尾连接到链表中的位置（索引从 0 开始）。 如果 pos 是 -1，则在该链表中没有环。注意：pos 不作为参数进行传递，仅仅是为了标识链表的实际情况。

如果链表中存在环，则返回 true 。 否则，返回 false 。

示例 1：
![](https://qxinhai.oss-cn-shenzhen.aliyuncs.com/hexo/20210327144340.png)
>输入：head = [3,2,0,-4], pos = 1 <br/>
输出：true <br/> 
解释：链表中有一个环，其尾部连接到第二个节点。 <br/>

示例 2：
![](https://qxinhai.oss-cn-shenzhen.aliyuncs.com/hexo/20210327144416.png)
>输入：head = [1,2], pos = 0 <br/>
输出：true <br/>
解释：链表中有一个环，其尾部连接到第一个节点。<br/>

示例 3：
![](https://qxinhai.oss-cn-shenzhen.aliyuncs.com/hexo/20210327144447.png)
>输入：head = [1], pos = -1 <br/>
输出：false <br/>
解释：链表中没有环。 <br/>


### 解法
```python
# Definition for singly-linked list.
class ListNode:
    def __init__(self, x):
        self.val = x
        self.next = None


class Solution:
    def hasCycle(self, head: ListNode) -> bool:
        if not head:
            return False
        if not head.next or not head.next.next:
            return False

        p = head
        q = head.next.next

        while True:
            if p == q:
                return True

            p = p.next
            if not q.next or not q.next.next:
                return False
            q = q.next.next


n3 = ListNode(3)
n2 = ListNode(2)
n0 = ListNode(0)
n4 = ListNode(4)
n3.next = n2
n2.next = n0
n0.next = n4
n4.next = n2

s = Solution()
print(s.hasCycle(n3))

```
