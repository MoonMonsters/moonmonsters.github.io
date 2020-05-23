title: django-rest-framework --- mixin模式
author: _Tao
tags:
  - django
  - django-rest-framework
categories:
  - python
date: 2020-05-23 19:08:00
---
假设存在这个三个类，Bird, Fish, Animal，分别具有不同的功能，像fly, swim, run等。现有某种虚拟Animal具有以上三种功能，那么便使用继承就可实现 。

```python
class Bird(object):

	def fly(self):
		print('can fly')


class Fish(object):

	def swim(self):
		print('can swim')


class Animal(object):

	def run(self):
		print('can run')


class VirtualAnimal(Bird, Fish, Animal):

	def can(self):
		self.fly()
		self.swim()
		self.run()


va = VirtualAnimal()
va.can()
```

<!-- more -->

可这么做就会出现一个 问题，当你看到VirtualAnimal这个类时，如果你事先不清楚它是Animal的话，单看这个类，你不会知道它到底是Bird, Fish 还是Animal。

如果使用mixin模式，即把类名改成MixinBird, MixinFish，那么类的继承就写成了

`class VirtualAnimal(MixinBird, MixinFish, Animal):`

虽然只是更改了类名，代码没有发生任何变化，但这是一种默认申明，该生物是Animal但具有fly(), swim()功能。

drf框架中，大量使用这种写法。