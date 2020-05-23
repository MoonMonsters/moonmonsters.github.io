title: (转)DRF中使用缓存drf-extensions
author: _Tao
tags:
  - django
  - django-rest-framework
categories:
  - python
date: 2020-05-23 19:25:00
---
(这个库很简单,直接转发别人的了.)<br/>

在开发某些网站的时候，比如有些不经常或者长时间不会变更的数据，但是，用户会经常查询的数据，为了避免多次查询数据库给数据库带来太大的压力，我们可以及将这些经常被查询的数据，在被查询一次之后，可以在存放在redis缓存中，下次再进行查询的时候，可以直接从redis进行获取，而不需要再查询存储数据的（例如mysql）数据库了；

<!-- more -->

在Django REST framework中使用缓存，可以通过drf-extensions扩展来实现。

### 安装
```python
pip install drf-extensions
```
最新版本是0.5.0,兼容py3以及drf3.9以上的版本,如果要在py2.7和drf3.6的版本上使用,需要安装`0.3.1`版本.

### 使用方法
#### 1.使用装饰器
可以在使用`rest_framework_extensions.cache.decorators`中的`cache_response`装饰器来装饰返回数据的类视图的对象方法：
```python
class CityView(views.APIView):
    @cache_response()
    def get(self, request, *args, **kwargs):
```
`cache_response`装饰器可以接收两个参数：
```python
@cache_response(timeout=60*60, cache='default')
```
timeout 缓存时间<br/>
cache 缓存使用的Django缓存后端（即CACHES配置中的键名称）<br/>
如果在使用cache_response装饰器时未指明timeout或者cache参数，则会使用配置文件中的默认配置，可以通过在settings.py中进行全局方法配置：
```python
# DRF扩展 
REST_FRAMEWORK_EXTENSIONS = { 
    # 缓存时间 
    'DEFAULT_CACHE_RESPONSE_TIMEOUT': 60 * 60, 
    # 缓存存储 
    'DEFAULT_USE_CACHE': 'default',
}
```
注意，cache_response装饰器既可以装饰在类视图中的get方法上，也可以装饰在REST framework扩展类提供的list或retrieve方法上。使用cache_response装饰器无需使用method_decorator进行转换。

#### 使用drf-extensions提供的扩展类
drf-extensions扩展对于缓存提供了三个扩展类：

    ListCacheResponseMixin
    用于缓存返回列表数据的视图，与ListModelMixin扩展类配合使用，实际是为list方法添加了cache_response装饰器
    
    RetrieveCacheResponseMixin
    用于缓存返回单一数据的视图，与RetrieveModelMixin扩展类配合使用，实际是为retrieve方法添加了cache_response装饰器
    
    CacheResponseMixin
    为视图集同时补充List和Retrieve两种缓存，与ListModelMixin和RetrieveModelMixin一起配合使用。

三个扩展类都是在rest_framework_extensions.cache.mixins中。

这个扩展类在使用的时候，只要需要进行缓存处理的视图函数在继承的时候，继承相关的扩展类，在全部的settings中进行缓存有效期和缓存使用的后端即可：

```python
class CityView(CacheResponseMixin, ReadOnlyModelViewSet):
	pass

```

```python
# DRF扩展 
REST_FRAMEWORK_EXTENSIONS = { 
# 缓存时间 
	'DEFAULT_CACHE_RESPONSE_TIMEOUT': 60 * 60, 
# 缓存存储 
	'DEFAULT_USE_CACHE': 'default', 
}
```

### 参考
```html
https://blog.csdn.net/gymaisyl/article/details/84452994
https://github.com/chibisov/drf-extensions/releases
http://chibisov.github.io/drf-extensions/docs/#0-3-1
```