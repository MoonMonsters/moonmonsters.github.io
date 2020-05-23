title: leetcode --- 有效的括号
author: _Tao
tags: []
categories: 
	- leetcode
date: 2020-05-23 17:27:00
---
[有效的括号](https://leetcode-cn.com/problems/valid-parentheses/)
<br/>
给定一个只包括 ‘(‘，’)’，’{‘，’}’，’[‘，’]’ 的字符串，判断字符串是否有效。

有效字符串需满足：

左括号必须用相同类型的右括号闭合。
左括号必须以正确的顺序闭合。
注意空字符串可被认为是有效字符串。

<!-- more -->

<br/>
<br/>

….没看清题目，以为是输入任意的字符串，然后写了以下做法.
```python
def isValid(self, s: str) -> bool:
	alist = []
	left = '{[('
	right = ')]}'
	for ch in s:
		if ch in left:
			alist.append(ch)
		elif alist:
			ch2 = alist[-1]
			if (ch2 == '(' and ch == ')') or (ch2 == '[' and ch == ']') or (ch2 == '{' and ch == '}'):
				alist.pop()
			elif ch in right:
				return False
		elif ch in right:
			alist.append(ch)
	return len(alist) == 0
```

写的很乱，但也通过了。
看清楚题目后，只会输入括号，那么题目就变得很简单了.
只需一直循环，然后删除掉成对的括号就行了。
```python
def isValid2(self, s: str) -> bool:
	while '()' in s or '{}' in s or '[]' in s:
		s = s.replace('()', '').replace('[]', '').replace('{}', '')

	return s == ''
```
上面的写法很简单，但耗时太长，可以改成以下写法，虽然相对于其他语言的耗时，仍旧是个dd。
```python
def isValid(self, s: str) -> bool:
	left = '{[('
	right = ')]}'
	alist = []

	for ch in s:
	   if ch in left or not alist:
		   if not alist and ch in right:
			   return False
		   alist.append(ch)
	   else:
		   ch2 = alist[-1]
		   if (ch2 == '(' and ch == ')') or \
				   (ch2 == '[' and ch == ']') or \
				   (ch2 == '{' and ch == '}'):
			   alist.pop()
		   else:
			   return False

	   return len(alist) == 0
```