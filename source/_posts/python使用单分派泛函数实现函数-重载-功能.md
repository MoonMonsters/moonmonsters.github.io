title: python使用单分派泛函数实现函数`重载`功能
author: _Tao
tags: []
categories:
  - python
date: 2020-05-23 19:06:00
---
众所周知,python中没有类似Java中的函数重载功能,当定义多个函数名一样但参数不一样的函数时,最后一个会前面前的同名函数覆盖.

<!-- more -->

### 模块中使用
- 我们使用singledispatch装饰器来实现"重载"功能
```python
from functools import singledispatch, partial
	
@singledispatch
def pprint(obj):
	return NotImplemented
```

- 使用`@<主函数>.register(type)`来装饰同名函数.分派函数可以有任意多个参数,但是具体调用哪一部分只由第一个参数类型决定,也就是`register(type)`中的type决定. 

- 对于单分派而言,函数名无关紧要,但不能同名就是了.
```python
@pprint.register(str)
def _pprint(obj):
	print(f'--->str类型: {obj}')

@pprint.register(Integral)
def _(obj):
	print(f'-->int类型: {obj}')
```

- 函数参数可以有多个,但如果有多个同类型的单分派函数,最后一个会覆盖前面的,在调用下面的函数时,必须传入两个参数,第一个是类型,第二个随意
```python
@pprint.register(str)
def _pprint(obj):
	print(f'--->str类型: {obj}')

@pprint.register(str)
def _pprint(obj1, obj2):
	print(f'--->str类型,两个参数: {obj1}, {obj2}')
```

- 可以识别的类型随意
```python
from numbers import Integral

class A(object):
	def __str__(self):
		return str(A.__name__)

@pprint.register(Integral)
def _(obj):
	print(f'-->Integral类型: {obj}')
	
@pprint.register(A)
def _(obj):
	print(f'自定义类型: {obj}')
```

- 也可以同时注册多种类型
```python
@pprint.register(float)
@pprint.register(dict)
def _(obj):
	print(f'{obj}.type = ' + str(type(obj)))
```

- 也可结合偏函数使用,当然,其实跟普通函数用法没什么区别
```python
printstr = partial(pprint, '固定第一个参数')
printstr('hello')
```


完整代码:
```python
#!/usr/bin/env python
# -*- coding: utf-8 -*-
# @Author: _Tao
# @Time: 19-5-15 21:48

from functools import singledispatch, partial
from numbers import Integral


class A(object):

	def __str__(self):
		return str(A.__name__)


@singledispatch
def pprint(obj):
	return NotImplemented


# @pprint.register(str)
# def _pprint(obj):
# 	print(f'--->str类型: {obj}')


@pprint.register(str)
def _pprint(obj1, obj2):
	print(f'--->str类型,两个参数: {obj1}, {obj2}')


@pprint.register(Integral)
def _(obj):
	print(f'--->int类型: {obj}')


@pprint.register(float)
@pprint.register(dict)
def _(obj):
	print(f'--->{obj}.type = ' + str(type(obj)))


@pprint.register(A)
def _(obj):
	print(f'--->自定义类型: {obj}')


printstr = partial(pprint, '固定第一个参数')

pprint('hello', 'world')
pprint(100)
pprint(1.34)
pprint({'name': 'chen'})
pprint(A())
printstr('hello')
```

输出:
```python
--->str类型,两个参数: hello, world
--->int类型: 100
--->1.34.type = <class 'float'>
--->{'name': 'chen'}.type = <class 'dict'>
--->自定义类型: A
--->str类型,两个参数: 固定第一个参数, hello
```

### 类中使用
只测试出这一种写法,无法使用self,不能添加staticmethod和classmethod,感觉实用意义不大.

```python
from functools import singledispatch


class CustomerPrint(object):

	@singledispatch
	def cprint(self, obj):
		print('default-->', obj)

	@cprint.register(str)
	def _(obj):
		print('str-->', obj)

	@cprint.register(int)
	def _(obj):
		print('int-->', obj)


CustomerPrint.cprint('h')
CustomerPrint.cprint(100)
```