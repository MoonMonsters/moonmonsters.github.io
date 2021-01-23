---
title: django - 读写锁
date: 2021-01-23 13:13:37
tags: [django]
categories:
  - python
---

### 代码来源

django1.11 版本, 完整源码路径`from django.utils.synch import RWLock`

实现方法主要是`RLock+Semaphore`结合使用.

在阅读`LocMemCache`本地内存缓存代码时, 注意到这一部分功能, 添加解释后以后可以用.

但在django3或者更之前的版本, 已经不再没有在`LocMemCache`中用`RWLock`, 而是直接使用python自带的`from threading import Lock`了.

<!-- more -->

### 源码及解释

```python
"""
Synchronization primitives:

    - reader-writer lock (preference to writers)

(Contributed to Django by eugene@lazutkin.com)
"""

import contextlib
import threading


class RWLock(object):
    """
    Classic implementation of reader-writer lock with preference to writers.

    Readers can access a resource simultaneously.
    Writers get an exclusive access.

    API is self-descriptive:
        reader_enters()
        reader_leaves()
        writer_enters()
        writer_leaves()
    """

    def __init__(self):
        # 可重入锁
        self.mutex = threading.RLock()
        # 读锁信号量
        self.can_read = threading.Semaphore(0)
        # 写锁信号量
        self.can_write = threading.Semaphore(0)
        # 正在进行读操作的数量
        self.active_readers = 0
        # 正在写操作的数量
        self.active_writers = 0
        # 等待操作
        self.waiting_readers = 0
        self.waiting_writers = 0

    def reader_enters(self):
        # 加锁
        with self.mutex:
            # 如果没有进行写操作
            if self.active_writers == 0 and self.waiting_writers == 0:
                # 那么读操作+1
                self.active_readers += 1
                # 激活读操作
                self.can_read.release()
            # 否则, 等待中的读操作+1
            # 等待写操作完成
            else:
                self.waiting_readers += 1
        # 获取读锁
        self.can_read.acquire()

    def reader_leaves(self):
        # 释放
        with self.mutex:
            # 读操作数-1
            self.active_readers -= 1
            # 没有读操作, 并且有等待中的写操作
            if self.active_readers == 0 and self.waiting_writers != 0:
                # 释放写操作的锁
                self.active_writers += 1
                self.waiting_writers -= 1
                self.can_write.release()

    @contextlib.contextmanager
    def reader(self):
        # contextlib.contextmanager装饰器的作用
        # 当代码被 with self.reader() 修饰时, 先执行yield之前的代码, 再执行代码块, 最后执行yield后的代码
        # 以此完成先获取锁再释放锁的操作
        # 获取锁
        self.reader_enters()
        try:
            yield
        finally:
            # 释放锁
            self.reader_leaves()

    def writer_enters(self):
        with self.mutex:
            # 释放写锁
            if self.active_writers == 0 and self.waiting_writers == 0 and self.active_readers == 0:
                self.active_writers += 1
                self.can_write.release()
            else:
                self.waiting_writers += 1
        self.can_write.acquire()

    def writer_leaves(self):
        with self.mutex:
            self.active_writers -= 1
            # 释放写锁
            if self.waiting_writers != 0:
                self.active_writers += 1
                self.waiting_writers -= 1
                self.can_write.release()
            # 释放读锁
            elif self.waiting_readers != 0:
                t = self.waiting_readers
                self.waiting_readers = 0
                self.active_readers += t
                while t > 0:
                    self.can_read.release()
                    t -= 1

    @contextlib.contextmanager
    def writer(self):
        self.writer_enters()
        try:
            yield
        finally:
            self.writer_leaves()

```



### 用法

```python
lock = RWLock()
with lock.reader():
	pass
with lock.writer():
	pass
```