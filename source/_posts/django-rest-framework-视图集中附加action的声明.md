title: django-rest-framework --- 视图集中附加action的声明
author: _Tao
tags:
  - django
  - django-rest-framework
categories:
  - python
date: 2020-05-23 19:16:00
---
在视图集(`ViewSet`)中，如果想要让Router自动帮助我们为自定义的动作生成路由信息，需要使用rest_framework.decorators.action装饰器。
以action装饰器装饰的方法名会作为action动作名，与list、retrieve等同。

`@action`可以接收四个参数: 

- methods: 声明该action对应的请求方式，列表传递
- detail: 声明该action的路径是否与单一资源对应，及是否是xxx/<pk>/action方法名/
	- True 表示路径格式是xxx/<pk>/action方法名/
	- False 表示路径格式是xxx/action方法名/
- url_path: 重设访问该函数的路径,默认是通过函数名`func.__name__`来访问
- url_name: 改函数对应的name,可用来设置跳转链接,例如revese('')等.

<!-- more -->

示例
```python
class BBSIndex(HandleExceptionMixin,
               mixins.ListModelMixin,
               mixins.RetrieveModelMixin,
               viewsets.GenericViewSet):
...
...
...
    @action(methods=['GET'], detail=True, url_path='test')
    def test_data(self, request, *args, **kwargs):
        return Response({'message': 'test action'})
```

```python
from django.conf.urls import url, include
from rest_framework import routers

router = routers.DefaultRouter()
router.register(r'all', views.BBSIndex, base_name='articles')

url(r'', include(router.urls)),
```

如果没有设置url_path的话,那么访问该函数的方式为:
```
http://192.168.0.60:10000/bbs/all/13/test_data/
```
而设置了url_path,则变成了
```
http://192.168.0.60:10000/bbs/all/13/test/
```