title: leetcode --- 146. LRU 缓存机制
author: _Tao
tags: []
categories:
  - leetcode
date: 2021-03-27 14:46:00
---
### 题目

[LRU 缓存机制](https://leetcode-cn.com/problems/lru-cache)

运用你所掌握的数据结构，设计和实现一个  LRU (最近最少使用) 缓存机制 。
实现 LRUCache 类：

LRUCache(int capacity) 以正整数作为容量 capacity 初始化 LRU 缓存
int get(int key) 如果关键字 key 存在于缓存中，则返回关键字的值，否则返回 -1 。
void put(int key, int value) 如果关键字已经存在，则变更其数据值；如果关键字不存在，则插入该组「关键字-值」。当缓存容量达到上限时，它应该在写入新数据之前删除最久未使用的数据值，从而为新的数据值留出空间。

示例：
> 输入
["LRUCache", "put", "put", "get", "put", "get", "put", "get", "get", "get"] <br/>
[[2], [1, 1], [2, 2], [1], [3, 3], [2], [4, 4], [1], [3], [4]] <br/>
输出 <br/>
[null, null, null, 1, null, -1, null, -1, 3, 4] <br/>
解释 <br/>
LRUCache lRUCache = new LRUCache(2);<br/>
lRUCache.put(1, 1); // 缓存是 {1=1}<br/>
lRUCache.put(2, 2); // 缓存是 {1=1, 2=2}<br/>
lRUCache.get(1);    // 返回 1<br/>
lRUCache.put(3, 3); // 该操作会使得关键字 2 作废，缓存是 {1=1, 3=3}<br/>
lRUCache.get(2);    // 返回 -1 (未找到)<br/>
lRUCache.put(4, 4); // 该操作会使得关键字 1 作废，缓存是 {4=4, 3=3}<br/>
lRUCache.get(1);    // 返回 -1 (未找到)<br/>
lRUCache.get(3);    // 返回 3<br/>
lRUCache.get(4);    // 返回 4<br/>


### 解法
```python
from collections import OrderedDict


class LRUCache:

    def __init__(self, capacity: int):
        self._order = OrderedDict()
        self._capacity = capacity

    def get(self, key: int) -> int:
        if key not in self._order.keys():
            return -1

        v = self._order.pop(key)
        self._order[key] = v

        return v

    def put(self, key: int, value: int) -> None:
        if key not in self._order.keys() and len(self._order.keys()) >= self._capacity:
            self._order.popitem(False)

        if key in self._order.keys():
            self._order.pop(key)
        self._order[key] = value


s = LRUCache(2)
s.put(1, 1)
s.put(2, 2)

```