title: Leetcode --- LRU缓存机制
author: _Tao
tags: []
categories:
  - Leetcode
date: 2020-05-23 17:54:00
---
[LRU缓存机制](https://leetcode-cn.com/problems/lru-cache/)

运用你所掌握的数据结构，设计和实现一个  LRU (最近最少使用) 缓存机制。它应该支持以下操作： 获取数据 get 和 写入数据 put 。

获取数据 get(key) - 如果密钥 (key) 存在于缓存中，则获取密钥的值（总是正数），否则返回 -1。
写入数据 put(key, value) - 如果密钥不存在，则写入其数据值。当缓存容量达到上限时，它应该在写入新数据之前删除最近最少使用的数据值，从而为新的数据值留出空间。

进阶:

你是否可以在 O(1) 时间复杂度内完成这两种操作？

示例:

LRUCache cache = new LRUCache( 2 /* 缓存容量 */ );

cache.put(1, 1);<br/>
cache.put(2, 2);<br/>
cache.get(1);       // 返回  1<br/>
cache.put(3, 3);    // 该操作会使得密钥 2 作废<br/>
cache.get(2);       // 返回 -1 (未找到)<br/>
cache.put(4, 4);    // 该操作会使得密钥 1 作废<br/>
cache.get(1);       // 返回 -1 (未找到)<br/>
cache.get(3);       // 返回  3<br/>
cache.get(4);       // 返回  4<br/>

<!-- more -->

使用python collections的OrderDict做的，就不需要再自己造轮子了...<br/>
一看跟其他语言的比较，性能真的是差的不止一点啊...<br/>
执行用时 : 144 ms, 在LRU Cache的Python3提交中击败了84.29% 的用户<br/>
内存消耗 : 21.8 MB, 在LRU Cache的Python3提交中击败了0.00% 的用户

```python
#!/usr/bin/env python
# -*- coding: utf-8 -*-
# @Author: _Tao
# @Time: 19-3-31 15:33

from collections import OrderedDict


class LRUCache:

	def __init__(self, capacity: int):
		self.__capacity = capacity
		self.__now_count = 0
		self.__cache = OrderedDict()

	def get(self, key: int) -> int:
		# 如果key值不存在，在返回-1
		if key not in self.__cache:
			return -1

		# 因为时有顺序的字典，那么要实现LRU缓存功能，就需要先删除，再插入
		# 那么最后一个元素就是最近使用的，最前面的元素就是最开始使用的
		# 当达到容量上限时，就删除最前面的元素
		value = self.__cache.pop(key)
		self.__cache[key] = value

		return value

	def put(self, key: int, value: int) -> None:
		# 如果要插入的元素不存在，并且容量已到上限时，就删除最开始使用的元素
		if key not in self.__cache and self.__now_count >= self.__capacity:
			# 容量-1
			self.__now_count -= 1
			# 删除最前面的元素
			self.__cache.popitem(last=False)

		# 如果要加入的元素已经存在了，那么就需要先删除
		if key in self.__cache:
			self.__now_count -= 1
			self.__cache.pop(key)

		self.__cache[key] = value
		self.__now_count += 1

	def print_length(self):
		print('now_count = {0}, cache.length = {1}'.format(self.__now_count, len(self.__cache)))


cache = LRUCache(2)

cache.put(1, 1)
cache.put(2, 2)
cache.print_length()

print(cache.get(1))
cache.put(3, 3)
print(cache.get(2))
cache.put(4, 4)
print(cache.get(1))
print(cache.get(3))
print(cache.get(4))

```