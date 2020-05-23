title: leetcode --- 环形链表
author: _Tao
tags: []
categories:
  - leetcode
date: 2020-05-23 17:16:00
---
[环形链表](https://leetcode-cn.com/problems/linked-list-cycle/)



给定一个链表，判断链表中是否有环。

为了表示给定链表中的环，我们使用整数 `pos` 来表示链表尾连接到链表中的位置（索引从 0 开始）。 如果 `pos` 是 `-1`，则在该链表中没有环。

**示例 1：**

```xml
输入：head = [3,2,0,-4], pos = 1
输出：true
解释：链表中有一个环，其尾部连接到第二个节点。
```

**示例 2：**

```xml
输入：head = [1,2], pos = 0
输出：true
解释：链表中有一个环，其尾部连接到第一个节点。
```

<!-- more -->

试了两种方法:

第一种用的是快慢指针，复杂度是60 ms\18.2 MB



```python
# class ListNode(object):
#     def __init__(self, x):
#         self.val = x
#         self.next = None

class Solution(object):

	def hasCycle(self, head):

		if not head or not head.next:
			return False

		fast = head
		slow = fast.next

		while fast != slow:
			try:
				fast = fast.next
				slow = slow.next.next
			except:
				return False

		return True
```



第二种试了下用一个 列表存下已经遍历过的节点，然后判断新的节点是否在列表中....但时间复杂度有点看不懂了，本来以为空间复杂度会很高，时间复杂度会低一点的 ： 1764 ms\18.1 MB 



```python
# class ListNode(object):
#     def __init__(self, x):
#         self.val = x
#         self.next = None

class Solution(object):
    def hasCycle(self, head):

		tmp = []
		while head:
			if head in tmp:
				break
			tmp.append(head)
			head = head.next
		else:
			return False

		return True
```