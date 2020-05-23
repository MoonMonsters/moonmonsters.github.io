title: leetcode --- 交替打印FooBar
author: _Tao
tags: []
categories:
  - leetcode
date: 2020-05-23 18:58:00
---
### 题目

[交替打印FooBar](https://leetcode-cn.com/problems/print-foobar-alternately/)

```java
class FooBar {
  public void foo() {
    for (int i = 0; i < n; i++) {
      print("foo");
    }
  }

  public void bar() {
    for (int i = 0; i < n; i++) {
      print("bar");
    }
  }
}
```
两个不同的线程将会共用一个 FooBar 实例。其中一个线程将会调用 foo() 方法，另一个线程将会调用 bar() 方法。

请设计修改程序，以确保 "foobar" 被输出 n 次。

示例1:
>输入: n = 1
输出: "foobar"
解释: 这里有两个线程被异步启动。其中一个调用 foo() 方法, 另一个调用 bar() 方法，"foobar" 将被输出一次。

示例2:
>输入: n = 2
输出: "foobarfoobar"
解释: "foobar" 将被输出两次。

<!-- more -->

### 代码

多线程中的一个题目，用Condition类完成。

```python
from threading import Thread, Condition


class FooBar:
    def __init__(self, n):
        self.n = n
        self.con = Condition()
        self.val = 'foo'

    def foo(self, printFoo: 'Callable[[], None]') -> None:

        for i in range(self.n):
            # printFoo() outputs "foo". Do not change or remove this line.
            with self.con:
                if self.val != 'foo':
                    self.con.wait()
                printFoo()
                self.val = 'bar'
                self.con.notify_all()

    def bar(self, printBar: 'Callable[[], None]') -> None:

        for i in range(self.n):
            # printBar() outputs "bar". Do not change or remove this line.
            with self.con:
                if self.val != 'bar':
                    self.con.wait()

                printBar()
                self.val = 'foo'
                self.con.notify_all()


def _printFoo():
    print('foo', end='')


def _printBar():
    print('bar')


foo = FooBar(10)
thrs = list()
thrs.append(Thread(target=foo.foo, args=(_printFoo,)))
thrs.append(Thread(target=foo.bar, args=(_printBar,)))

for th in thrs:
    th.start()

for th in thrs:
    th.join()

```