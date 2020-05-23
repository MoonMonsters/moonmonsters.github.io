title: leetcode --- 打印零与奇偶数
author: _Tao
tags: []
categories:
  - leetcode
date: 2020-05-23 19:00:00
---
### 题目

[打印零与奇偶数](https://leetcode-cn.com/problems/print-zero-even-odd/)

假设有这么一个类：
```java
class ZeroEvenOdd {
  public ZeroEvenOdd(int n) { ... }      // 构造函数
  public void zero(printNumber) { ... }  // 仅打印出 0
  public void even(printNumber) { ... }  // 仅打印出 偶数
  public void odd(printNumber) { ... }   // 仅打印出 奇数
}

```
相同的一个 ZeroEvenOdd 类实例将会传递给三个不同的线程：

    线程 A 将调用 zero()，它只输出 0 。
    线程 B 将调用 even()，它只输出偶数。
    线程 C 将调用 odd()，它只输出奇数。

每个线程都有一个 printNumber 方法来输出一个整数。请修改给出的代码以输出整数序列 010203040506... ，其中序列的长度必须为 2n。
示例1:
输入：n = 2<br/>
输出："0102"<br/>
说明：三条线程异步执行，其中一个调用 zero()，另一个线程调用 even()，最后一个线程调用odd()。正确的输出为 "0102"。
示例2:
输入：n = 5<br/>
输出："0102030405"

<!-- more -->

### 题解
```python
# !/usr/bin/env python
# -*- coding:utf-8 -*-
# __author__ = 'ChenTao'

import threading


class ZeroEvenOdd:
    BREAKPOINT = -1

    def __init__(self, n):
        self.n = n
        self.cur = 0
        self.index = 1
        self.condition = threading.Condition()

    def zero(self, printNumber) -> None:
        while True:
            with self.condition:
                # 判断是否等待唤醒
                while not self.__is_zero():
                    self.condition.wait()
                # 退出循环
                if self.cur == self.BREAKPOINT:
                    break
                # 打印
                printNumber(self.cur)
                # 计算下一次打印的数字
                self.__continue()
                # 唤醒
                self.condition.notify_all()

    def even(self, printNumber) -> None:
        while True:
            with self.condition:
                while not self.__is_even():
                    self.condition.wait()
                if self.cur == self.BREAKPOINT:
                    break
                printNumber(self.cur)
                self.__continue()
                self.condition.notify_all()

    def odd(self, printNumber) -> None:
        while True:
            with self.condition:
                while not self.__is_odd():
                    self.condition.wait()
                if self.cur == self.BREAKPOINT:
                    break
                printNumber(self.cur)
                self.__continue()
                self.condition.notify_all()

    def __is_zero(self):
        """
        打印数字0
        """
        return self.cur == 0 or self.cur == self.BREAKPOINT

    def __is_even(self):
        """
        打印奇数
        """
        return (self.cur != 0 and self.cur % 2 == 0) or self.cur == self.BREAKPOINT

    def __is_odd(self):
        """
        打印偶数
        """
        return (self.cur != 0 and self.cur % 2 == 1) or self.cur == self.BREAKPOINT

    def __continue(self):
        if self.index <= self.n:
            if self.cur == 0:
                self.cur = self.index
                self.index += 1
            else:
                self.cur = 0
        else:
            self.cur = self.BREAKPOINT


# 测试代码
def print_number(num):
    print(num, end='')


def func(ze: ZeroEvenOdd, fname: str):
    _fc = getattr(ze, fname)
    _fc(print_number)


def main():
    n = 8
    ze = ZeroEvenOdd(n)
    thrs = list()
    thrs.append(threading.Thread(target=func, args=(ze, 'zero')))
    thrs.append(threading.Thread(target=func, args=(ze, 'odd')))
    thrs.append(threading.Thread(target=func, args=(ze, 'even')))

    for th in thrs:
        th.start()

    for th in thrs:
        th.join()


if __name__ == '__main__':
    main()

```