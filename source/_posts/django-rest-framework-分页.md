title: Django-rest-framework --- 分页
author: _Tao
tags:
  - django
  - django-rest-framework
categories:
  - Python
date: 2020-05-23 19:32:00
---
### 介绍

drf框架的分页主要有三种：

1. 普通分页，看第n页，每页显示m条数据
2. 切割分页，在第n个位置，向后查看m条数据
3. 加密分页，与普通分页一样，不过对url中的请求页码进行加密

<!-- more -->

### 普通分页

#### 自定义分页类

```python
from rest_framework import pagination


class StudentPagination(pagination.PageNumberPagination):
	# 默认值 ，每一页的数量，如果没有size参数的话
	page_size = 8
	# 最大数量，即使 带上了size参数，也无法超过这个值
	max_page_size = 10
	# 通过GET请求获取每一页需要的数量，/?size=x的size参数
	page_size_query_param = 'size'
	# url中要查找的page参数，即/?page=x中的page参数
	page_query_param = 'page'
```

#### 视图

```python
class StudentView(generics.ListAPIView):

	def get_queryset(self):
		return Student.objects.all().order_by('-id')

	def list(self, request, *args, **kwargs):
        # 实例化我们定义的分页类
		pagination = StudentPagination()
        # 对实例化类进行传参控制
		students = pagination.paginate_queryset(self.get_queryset(), request=request, view=self)
        # 将分页后的对象作序列化
		serializer = StudentSerializer(students, many=True)

		return pagination.get_paginated_response(serializer.data)
```

#### url

```python
urlpatterns = [
	path('', views.StudentView.as_view())
]
```

在请求时，可以使用`http://127.0.0.1:8000/students/?page=10&size=3`，那么在使用时，会自动从链接中提取page和size的值了。此处的意思是，请求第10页的3个数据。



### 切割分页

#### 自定义分页类

```python
class StudentLimitOffsetPagination(pagination.LimitOffsetPagination):
	# 默认每一页显示多少条数据
	default_limit = 8
	# url中设置显示数据数量的参数
	limit_query_param = 'limit'
	# 从数据库中的第几条开始查询，查询limit条
	offset_query_param = 'offset'
	# 每一次请求，返回最大数量
	max_limit = 10
```

其他的写法跟普通分页无异。



### 加密分页

#### 自定义分页类

```python
class StudentCursorPagination(pagination.CursorPagination):
	# 查询参数
	cursor_query_param = 'cursor'
	# 排序方式
	ordering = '-id'
	# 每页查询数量
	page_size_query_param = 'size'
	# 每页大小
	page_size = 8
	# 最大限制
	max_page_size = 10
```



但返回数据却有不同，无法通过修改url来得到指定某页的数据，只能通过返回数据中的previous和next来得到上一页或者下一页的数据。

```json
{
    "next": "http://127.0.0.1:8000/students/?cursor=cD04NQ%3D%3D",
    "previous": "http://127.0.0.1:8000/students/?cursor=cj0xJnA9OTI%3D",
    "results": [
        {
            "id": 92,
            "name": "test91",
            "age": 101,
            "sex": 1,
            "height": 0.349139736956964,
            "weight": 105.24197499223212
        }
    ]
}
```