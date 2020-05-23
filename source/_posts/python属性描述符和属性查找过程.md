title: python属性描述符和属性查找过程
author: _Tao
tags: []
categories:
  - python
date: 2020-05-23 19:34:00
---
### 对象的自省机制
自省是通过一定的机制查询到对象的内部结构。
#### `dir(obj)`
dir是python提供的一个api函数，会返回一个列表，该列表中包含该对象（实例对象或者类对象）的所有的属性和方法（包括从父类中获取的）

<!-- more -->

#### `__dict__`
__dict__字典中存储的是对象或类的部分属性，键为属性名，值为属性值。
实例对象的__dict__中只存储跟实例对象相关的属性。
类对象的__dict__中存储着能和实例对象共享的属性和方法，类的__dict__中并不包含从父类继承的属性和方法。
#### `dir`和`__dict__`的区别
- dir是一个函数，返回的数据是一个list，只有属性名和函数名
- __dict__是一个字典，键为属性名\方法（函数）名，值是属性的值或者具体的方法（函数）
- dir用来寻找一个对象的所有的属性和方法，包括从父类中继承的

#### 示例
```python
class A(object):
	Aname = 'hello'

	def __init__(self):
		self.__age = 20

	@property
	def age(self):
		return self.__age

	@age.setter
	def age(self, _):
		self.__age = _

	def funA(self):
		print('--->funcA')


class B(A):
	Bname = 'world'

	def __init__(self):
		super(B, self).__init__()
		self.__sex = '男'

	def funcB(self):
		print('--->funB')

b = B()
```
`b.__dict__`
```
{'_A__age': 20, '_B__sex': '男'}
```

`B.__dict__`
```python
{'__module__': '__main__', 'Bname': 'world', '__init__': <function B.__init__ at 0x7fd2a753e378>, 'funcB': <function B.funcB at 0x7fd2a753e400>, '__doc__': None}
```
`dir(b)`
```python
['Aname', 'Bname', '_A__age', '_B__sex', '__class__', '__delattr__', '__dict__', '__dir__', '__doc__', '__eq__', '__format__', '__ge__', '__getattribute__', '__gt__', '__hash__', '__init__', '__init_subclass__', '__le__', '__lt__', '__module__', '__ne__', '__new__', '__reduce__', '__reduce_ex__', '__repr__', '__setattr__', '__sizeof__', '__str__', '__subclasshook__', '__weakref__', 'age', 'funA', 'funcB']
```
`dir(B)`
与`dir(b)`对比可以看出，实例对象的值在这个列表里面不存在。
```python
['Aname', 'Bname', '__class__', '__delattr__', '__dict__', '__dir__', '__doc__', '__eq__', '__format__', '__ge__', '__getattribute__', '__gt__', '__hash__', '__init__', '__init_subclass__', '__le__', '__lt__', '__module__', '__ne__', '__new__', '__reduce__', '__reduce_ex__', '__repr__', '__setattr__', '__sizeof__', '__str__', '__subclasshook__', '__weakref__', 'age', 'funA', 'funcB']
```

### `__getattr__`和`__getattribute__`魔法方法
`__getattr__`是在查找不到属性时调用
`__getattribute__`在访问对象时优先调用，不管属性是否存在，所以不建议重写该魔法方法

```python
class A(object):
	name = 'chen'

	def __init__(self):
		self.age = 20

	def __getattr__(self, item):
		print('__getattr__')
		return item

	def __getattribute__(self, item):
		print('__getattribute__')

		return super(A, self).__getattribute__(item)


a = A()
```

当调用存在的属性时打印：
`a.name`
```python
__getattribute__
chen
```
当调用不存在的属性时打印：
`a.sss`
```python
__getattribute__
__getattr__
sss
```
也就是说，不管调用的属性存在不存在，都会调用`__getattribute__`魔法方法；当调用的属性不存在时，`__getattribute__`会抛出异常，`AttributeError`，然后在内部调用`__getattr__`函数。
> 注意:
> 不能在`__getattribute__`方法中使用`self.xx`或者`self.__dict__[xxx]`或者`hasattr(self, xxx)`等操作，会陷入无限递归之中

### 属性描述符
`__get__`
`__set__`
`__delete__`
实现上面三个魔法函数之一就可称之为属性描述符，如果只实现了`__get__`则称为非数据属性描述符(no-data descriptor)，只有同时实现了`__get__`和`__set__`才称之为数据属性描述符(data descriptor)。

以下例子就是实现了一个int类型的数据属性描述符
```python
import numbers


class Integer(object):

	def __set__(self, instance, value):
		if not isinstance(value, numbers.Integral):
			raise ValueError('必须是int类型数据')

		print('__set__')
		self.__value = value

	def __get__(self, instance, owner):
		print('__get__')
		return self.__value

	def __delete__(self, instance):
		print('__delete__')
		del self.__value


class Test(object):
	age = Integer()
```

测试：
```python
t = Test()
t.age = 100
print(t.age)
del t.age
```
输出:
```python
__set__
__get__
100
__delete__
```

### 属性查找过程
```python
import numbers


class Integer(object):

	def __set__(self, instance, value):
		if not isinstance(value, numbers.Integral):
			raise ValueError('必须是int类型数据')

		print('__set__')
		self.__value = value

	def __get__(self, instance, owner):
		print('__get__')
		return self.__value

	def __delete__(self, instance):
		print('__delete__')
		del self.__value


class Test(object):
	age = Integer()

	def __init__(self):
		self.score = Integer()

	def __getattr__(self, item):
		print('__getattr__')
		return item

	def __getattribute__(self, item):
		print('__getattribute__')

		return super(Test, self).__getattribute__(item)


t = Test()
```

- 如果attr是数据属性描述符，并且是类对象属性，即代码中`age`属性，那么调用时，执行顺序: `__getattribute__` --> `__get__`
```python
t.age = 100
print(t.age)
```
输出
```python
__set__
__getattribute__
__get__
100
```
- 如果attr是数据属性描述符，但为对象实例的属性时，即`score`属性，执行顺序为就是调用`__getattribute__`取数据，即直接调用`self.__dict__`中取数据了，跟是否时数据属性描述符无关。
```python
t.score = 99
print(t.score)
```
输出:
```python
__getattribute__
99
```

- 如果attr存在，则执行顺序是`__geattribute__` --> `t.__dict__` --> `Test.__dict__`
- 如果attr不存在，则查找顺序是`__getattribute__` --> `t.__dict__` --> `Test.__dict__` --> `__getattr__`


### 参考
```python
https://www.cnblogs.com/xyz2b/p/10529068.html
https://www.cnblogs.com/xybaby/p/6270551.html
https://www.jianshu.com/p/885d59db57fc
https://blog.csdn.net/qq_26442553/article/details/82467777
https://www.cnblogs.com/cccy0/archive/2018/05/20/9063679.html
https://www.cnblogs.com/Vito2008/p/5280216.html
```