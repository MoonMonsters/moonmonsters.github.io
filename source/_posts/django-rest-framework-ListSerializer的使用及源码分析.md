title: django-rest-framework --- ListSerializer的使用及源码分析
author: _Tao
tags:
  - django
  - django-rest-framework
categories:
  - python
date: 2020-05-23 19:20:00
---
### 目的

想根据不同的情况，例如登录用户的身份，权限等，可添加或删除返回的数据中的字段。

最开始以为只需要定义一个序列化类然后重写data方法即可，但发现返回多条数据时，要删除的字段一直存在。翻看了官方文档，结合源码才发现还需要其他操作。

解决了这个问题，特简单 记录下。

<!-- more -->

### 使用

#### 序列化类

```python
from rest_framework import serializers

from demo.models import Student


class StudentListSerializer(serializers.ListSerializer):
    class Meta:
        model = Student
        fields = '__all__'

    def update(self, instance, validated_data):
        pass

    @property
    def data(self):
        _data = super(StudentListSerializer, self).data
        # 列表
        for d in _data:
            d.pop('id')

        return _data


class StudentSerializer(serializers.ModelSerializer):
    class Meta:
        model = Student
        # 重要
        # 当序列化操作为many时，调用的序列化类
        list_serializer_class = StudentListSerializer
        fields = '__all__'

    @property
    def data(self):
        # 单条数据
        _data = super(StudentSerializer, self).data
        # 移除掉id字段
        _data.pop('id')

        return _data

```



#### 视图类

```python
from rest_framework import mixins
from rest_framework import viewsets
from rest_framework import pagination

from demo.models import Student
from demo.serializers import StudentSerializer


class StudentView(mixins.ListModelMixin,
                  viewsets.GenericViewSet):
    serializer_class = StudentSerializer
    queryset = Student.objects.all()
    pagination_class = pagination.PageNumberPagination


class StudentDetailView(mixins.RetrieveModelMixin,
                        viewsets.GenericViewSet):
    serializer_class = StudentSerializer
    queryset = Student.objects.all()

```



### 源码

#### ListModelMixin

在视图中，使用了mixin模式，继承了`ListModelMixin`类，该类下，在返回序列化对象时，会传入`many=True`。

```python
class ListModelMixin(object):
    """
    List a queryset.
    """
    def list(self, request, *args, **kwargs):
        queryset = self.filter_queryset(self.get_queryset())

        page = self.paginate_queryset(queryset)
        if page is not None:
            # 需要返回一组数据，需要传入many=True参数
            serializer = self.get_serializer(page, many=True)
            return self.get_paginated_response(serializer.data)

        serializer = self.get_serializer(queryset, many=True)
        return Response(serializer.data)
```



#### ger_serializer

这个没什么好说的，就是返回在View中定义的视图类对象，并传入参数。

```python
serializer_class = StudentSerializer
```



```python
def get_serializer(self, *args, **kwargs):
    serializer_class = self.get_serializer_class()
    kwargs['context'] = self.get_serializer_context()
    return serializer_class(*args, **kwargs)

def get_serializer_class(self):
    assert self.serializer_class is not None, (
        "'%s' should either include a `serializer_class` attribute, "
        "or override the `get_serializer_class()` method."
        % self.__class__.__name__
    )

    return self.serializer_class
```



#### data()

在返回数据时，都是使用序列化对象调用data方法，像`serializer.data`。

所以要想添加或删除返回的数据字段时，就需要重写data方法了。

`ModelSerializer`类下没有data方法;

`ListSerializer`下的data方法:

```python
@property
def data(self):
    ret = super(ListSerializer, self).data
    return ReturnList(ret, serializer=self)
```

最终都是调用父类`BaseSerializer`下的data方法。

```python
@property
def data(self):
    if hasattr(self, 'initial_data') and not hasattr(self, '_validated_data'):
        msg = (
            'When a serializer is passed a `data` keyword argument you '
            'must call `.is_valid()` before attempting to access the '
            'serialized `.data` representation.\n'
            'You should either call `.is_valid()` first, '
            'or access `.initial_data` instead.'
        )
        raise AssertionError(msg)

    if not hasattr(self, '_data'):
        if self.instance is not None and not getattr(self, '_errors', None):
            self._data = self.to_representation(self.instance)
        elif hasattr(self, '_validated_data') and not getattr(self, '_errors', None):
            self._data = self.to_representation(self.validated_data)
        else:
            self._data = self.get_initial()
    return self._data
```



#### \_\_new\_\_

在创建序列化对象时，会判断是否传入了`many`参数，如果为True的话，就会调用`many_init`方法，最终执行的代码是：

```python
# 判断Serializer类中是否定义了Meta类
meta = getattr(cls, 'Meta', None)
# 如果Meta中没有设置list_serializer_class字段，那么就默认使用ListSerializer序列化类
list_serializer_class = getattr(meta, 'list_serializer_class', ListSerializer)
```

再回到`StudentListSerializer`中，如果要想重写`data`方法生效，那么就需要设置该字段 了

```python
class Meta:
	model = Student
	list_serializer_class = StudentListSerializer
	fields = '__all__'
```



```python
def __new__(cls, *args, **kwargs):
    if kwargs.pop('many', False):
        return cls.many_init(*args, **kwargs)
    return super(BaseSerializer, cls).__new__(cls, *args, **kwargs)


@classmethod
def many_init(cls, *args, **kwargs):
    allow_empty = kwargs.pop('allow_empty', None)
    child_serializer = cls(*args, **kwargs)
    list_kwargs = {
        'child': child_serializer,
    }
    if allow_empty is not None:
        list_kwargs['allow_empty'] = allow_empty
    list_kwargs.update({
        key: value for key, value in kwargs.items()
        if key in LIST_SERIALIZER_KWARGS
    })
    # 如果序列化对象中没有设置list_serializer_class属性，那么就使用默认的ListSerializer序列化器
    meta = getattr(cls, 'Meta', None)
    list_serializer_class = getattr(meta, 'list_serializer_class', ListSerializer)
    return list_serializer_class(*args, **list_kwargs)
```



### 参考

[ListSerializer](https://www.django-rest-framework.org/api-guide/serializers/#listserializer)