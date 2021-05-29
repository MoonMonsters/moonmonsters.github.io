title: Python -- 四种单例模式写法
author: _Tao
tags: []
categories:
  - Python
date: 2020-05-23 19:11:00
---
```python
"""
通过单例模式，可以保证系统中的这个类只产生一个对象且容易被外界访问，方便控制个例个数以及节省资源
"""
# 1
class Singleton(object):
    def __new__(cls,*args,**kwargs):
        """
        确保每次通过__new__函数，返回的对象都是同一个即可
        """
        if not hasattr(cls,'_instance'):
            #orig = super(Singleton,cls)
            #cls._instance = orig.__new__(cls,*args,**kwargs)
            # or
            cls._instance = super().__new__(cls,*args,**kwargs)
        return cls._instance

class A(Singleton):
    num = 0

a1 = A()
a2 = A()
a3 = A()

print(id(a1),id(a2),id(a3))
a1.num = 100
print(a2.num,a3.num,A.num)

print('-' * 80)

#2
class Singleton2(object):
    _state = {}
    def __new__(cls,*args,**kwargs):
        single = super().__new__(cls,*args,**kwargs)
        single.__dict__ = cls._state

        return single

class B(Singleton2):
    num = 0

b1 = B()
b2 = B()
b3 = B()
# 每次返回的对象都不一样
print(id(b1),id(b2),id(b3))

b1.num = 100
print(b2.num,b3.num,B.num)
# 但贡献了__dict__的魔法方法，即具有的属性，数据都是一样的，一个对象修改，其他对象跟着修改
print(b1.__dict__,b2.__dict__,b3.__dict__)
print(b1._state)

print('-' * 80)

# 3
"""
跟第一种方法效果一样，都是只生成一个对象
使用一个dict存储类对象，在使用装饰器时先判断cls是否已经存在，如果存在则取出，否则创建存入再取出
就能确保每次都是只使用同一个对象
"""
def singleton(cls):
    instances = {}
    def getinstance(*args,**kwargs):
        if cls not in instances:
            instances[cls] = cls(*args,**kwargs)

        return instances[cls]
    return getinstance

@singleton
class C(object):
    num = 0

c1 = C()
c2 = C()
c3 = C()
print(id(c1),id(c2),id(c3))

print('-' * 80)

# 4
"""
模块导入，在python中，模块是天然的单例模式
"""
"""
# 在 single.py 中的类
class Singleton4(object):
    num = 0
    def foo(self):
        print(self.num)

single = Singleton4()
"""
from single import single,Singleton4

# 但仅限于把已经创建好的实例对象导入进来
single.num = 100
single.foo()

# 如果是导入了类，然后创建了实例对象，那么跟普通类没有区别
s1 = Singleton4()
s2 = Singleton4()
s3 = Singleton4()
print(id(s1),id(s2),id(s3))

s1.num = 500
print(s2.num,s3.num,Singleton4.num)
```