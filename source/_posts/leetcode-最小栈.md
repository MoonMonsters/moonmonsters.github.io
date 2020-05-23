title: leetcode --- 最小栈
author: _Tao
tags: []
categories:
  - leetcode
date: 2020-05-23 19:00:00
---
### 题目

[最小栈](https://leetcode-cn.com/problems/min-stack/)

<!-- more -->

设计一个支持 push，pop，top 操作，并能在常数时间内检索到最小元素的栈。

    push(x) -- 将元素 x 推入栈中。
    pop() -- 删除栈顶的元素。
    top() -- 获取栈顶元素。
    getMin() -- 检索栈中的最小元素。

```python
MinStack minStack = new MinStack();
minStack.push(-2);
minStack.push(0);
minStack.push(-3);
minStack.getMin();   --> 返回 -3.
minStack.pop();
minStack.top();      --> 返回 0.
minStack.getMin();   --> 返回 -2.

```



### 解法

#### 1. 第一种解法: list + min

第一种解法就是利用`list`存放数据，然后在获取最小值的时，使用内置函数`min`。

PS:没考虑异常的情况了，例如空列表的处理之类的。

```python
class MinStack:
    def __init__(self):
        """
        initialize your data structure here.
        """
        self.__stack = []

    def push(self, x: int) -> None:
        self.__stack.append(x)

    def pop(self) -> None:
        self.__stack.pop()

    def top(self) -> int:
        return self.__stack[-1]

    def getMin(self) -> int:
        return min(self.__stack)


import random

stack = MinStack()
nums = [random.randint(0, 100) for _ in range(10)]
print('nums: ' + str(nums))
for num in nums:
    stack.push(num)

print('min: ' + str(stack.getMin()))
stack.pop()
stack.pop()
stack.pop()
stack.pop()
print('min: ' + str(stack.getMin()))
print('top: ' + str(stack.top()))

```

leetcode的算法复杂度，用时`988ms`



#### 第二种解法: 双deque

这种解法利用了python自带的 deque数据结构，用空间换时间来解的 。

```python
from collections import deque


class MinStack:
    def __init__(self):
        """
        initialize your data structure here.
        """
        self.__stack = deque()
        self.__min_stack = deque()

    def push(self, x: int) -> None:
        self.__stack.append(x)
        if not self.__min_stack or self.__min_stack[-1] >= x:
            self.__min_stack.append(x)

    def pop(self) -> None:
        x = self.__stack.pop()
        if self.__min_stack and self.__min_stack[-1] == x:
            self.__min_stack.pop()

    def top(self) -> int:
        return self.__stack[-1]

    def getMin(self) -> int:
        return self.__min_stack[-1]


import random

stack = MinStack()
nums = [random.randint(0, 100) for _ in range(10)]
print('nums: ' + str(nums))
for num in nums:
    stack.push(num)

print('min: ' + str(stack.getMin()))
stack.pop()
stack.pop()
stack.pop()
stack.pop()
print('min: ' + str(stack.getMin()))
print('top: ' + str(stack.top()))

```

leetcode上通过时间是`148ms`