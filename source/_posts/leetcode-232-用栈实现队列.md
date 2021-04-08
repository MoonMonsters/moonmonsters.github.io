title: leetcode --- 232. 用栈实现队列
author: _Tao
tags: []
categories:
  - leetcode
date: 2021-03-27 14:56:00
---
### 题目

[用栈实现队列](https://leetcode-cn.com/problems/implement-queue-using-stacks)

请你仅使用两个栈实现先入先出队列。队列应当支持一般队列支持的所有操作（push、pop、peek、empty）：

实现 MyQueue 类：
- void push(int x) 将元素 x 推到队列的末尾
- int pop() 从队列的开头移除并返回元素
- int peek() 返回队列开头的元素
- boolean empty() 如果队列为空，返回 true ；否则，返回 false


### 解法
```python
class MyQueue:

    def __init__(self):
        """
        Initialize your data structure here.
        """
        self._in = list()
        self._out = list()

    def push(self, x: int) -> None:
        """
        Push element x to the back of queue.
        """

        self._in.append(x)

    def _transfer(self):
        while self._in:
            x = self._in.pop()
            self._out.append(x)

    def pop(self) -> int:
        """
        Removes the element from in front of queue and returns that element.
        """
        if not self._out:
            self._transfer()
        return self._out.pop()

    def peek(self) -> int:
        """
        Get the front element.
        """
        if not self._out:
            self._transfer()
        return self._out[-1]

    def empty(self) -> bool:
        """
        Returns whether the queue is empty.
        """
        return len(self._in) == 0 and len(self._out) == 0


# Your MyQueue object will be instantiated and called as such:
obj = MyQueue()
print(obj.empty())

```