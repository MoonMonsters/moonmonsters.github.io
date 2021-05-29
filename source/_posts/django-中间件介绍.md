title: Django --- 中间件介绍
author: _Tao
tags:
  - django
categories:
  - Python
date: 2020-05-23 19:16:00
---
### 概念
在http请求到达函数之前和函数return之后,django会根据自己的规则在合适的时机执行中间件中相应的方法.因为改变的是全局,所以谨慎使用.

中间件是一个与django的请求/响应处理相关的框架,是一种轻的,低层次的"插件"系统,用于django的全局的输入/输出.

如果你想修改请求，例如被传送到view中的HttpRequest对象。
或者你想修改view返回的HttpResponse对象，这些都可以通过中间件来实现。
可能你还想在view执行之前做一些操作，这种情况就可以用 middleware来实现。

<!-- more -->

### 流程
流程图:
![流程](https://qxinhai.oss-cn-shenzhen.aliyuncs.com/pic/55-1.jpg)

在django==1.8.2的django.core.handlers.base.BaseHandler类中,有这么个函数load_middleware,其中有段代码
```python
if hasattr(mw_instance, 'process_request'):
    request_middleware.append(mw_instance.process_request)
if hasattr(mw_instance, 'process_view'):
    self._view_middleware.append(mw_instance.process_view)
if hasattr(mw_instance, 'process_template_response'):
    self._template_response_middleware.insert(0,
mw_instance.process_template_response)
if hasattr(mw_instance, 'process_response'):
    self._response_middleware.insert(0, mw_instance.process_response)
if hasattr(mw_instance, 'process_exception'):
    self._exception_middleware.insert(0, mw_instance.process_exception)
```

但在django==2.0中,这部分代码被拆分到了两个类中

django.util.deprecation:
```python
class MiddlewareMixin:
    def __init__(self, get_response=None):
        self.get_response = get_response
        super().__init__()
 
    def __call__(self, request):
        response = None
        if hasattr(self, 'process_request'):
            response = self.process_request(request)
        response = response or self.get_response(request)
        if hasattr(self, 'process_response'):
            response = self.process_response(request, response)
        return response
```

以及, django.core.handlers.base.BaseHandler.load_middleware函数中:
```python
if hasattr(mw_instance, 'process_view'):
    self._view_middleware.insert(0, mw_instance.process_view)
if hasattr(mw_instance, 'process_template_response'):
self._template_response_middleware.append(mw_instance.process_template_response)
if hasattr(mw_instance, 'process_exception'):
self._exception_middleware.append(mw_instance.process_exception)
```

大致流程是,在函数没有返回值的情况下,会按照settings.py中的中间件的顺序先后执行process_request和process_view函数,但执行完视图函数后,会按照逆向顺序执行process_exception,
process_template_response 和 process_response.
创建完django项目后,会在settings.py中自动添加以下中间件(django1.8.0中是MIDDLEWARE_CLASSES)
```python
MIDDLEWARE = [
    'django.middleware.security.SecurityMiddleware',
    'django.contrib.sessions.middleware.SessionMiddleware',
    'django.middleware.common.CommonMiddleware',
    'django.middleware.csrf.CsrfViewMiddleware',
    'django.contrib.auth.middleware.AuthenticationMiddleware',
    'django.contrib.messages.middleware.MessageMiddleware',
    'django.middleware.clickjacking.XFrameOptionsMiddleware',
]
```
那么在没有任何返回值以及异常的情况下,
process_request的执行顺序是
SecurityMiddleware-->SessionMiddleware-->CommonMiddleware-->...
process_view同上,
但后三者的执行顺序会是
XFrameOptionMiddleware-->MessageMiddleware-->AuthenticationMiddleware-->...
其实从代码就可以大致看出来,前两者用的是`append`,后三者用的是`insert`.

### 自定义中间件
按需要实现5个函数:
```python
process_request(self,request)
process_view(self, request, view_func, view_args, view_kwargs)
process_template_response(self,request,response)
process_exception(self, request, exception)
process_response(self, request, response)
```
以上方法的返回值可以是None或一个HttpResponse对象，如果是None，则继续按django定义的规则向后继续执行，如果是HttpResponse对象，则直接将该对象返回给用户。

#### 代码
md1.py
```python
try:
    from django.utils.deprecation import MiddlewareMixin
except ImportError:
    MiddlewareMixin = object
 
 
class MiddlewareTest1(MiddlewareMixin):
 
    def process_request(self, request):
        print('md1.process_request')
 
    def process_view(self, request, view_func, view_args, view_kwargs):
        print('md1.process_view')
 
    def process_template_response(self, request, response):
        print('md1.process_template_response')
 
        return response
 
    def process_exception(self, request, exception):
        print('md1.process_exception')
 
    def process_response(self, request, response):
        print('md1.process_response')
 
        return response
```

md2.py
```python
try:
    from django.utils.deprecation import MiddlewareMixin
except ImportError:
    MiddlewareMixin = object
 
 
class MiddlewareTest2(MiddlewareMixin):
 
    def process_request(self, request):
        print('md2.process_request')
 
    def process_view(self, request, view_func, view_args, view_kwargs):
        print('md2.process_view')
 
    def process_template_response(self, request, response):
        print('md2.process_template_response')
 
        return response
 
    def process_exception(self, request, exception):
        print('md2.process_exception')
 
    def process_response(self, request, response):
        print('md2.process_response')
 
        return response
```

views.py
```python
from django.shortcuts import render
 
from rest_framework.views import APIView
from rest_framework.response import Response
 
 
class IndexApiView(APIView):
 
    def get(self, request, *args, **kwargs):
        print('IndexApiView.get')
 
        return render(request, 'demo/index.html')
        # return Response({'result': True})
```

需要注意几点:
1.
MiddlewareMixin在django==1.8.0中是没有的,所以需要捕获异常,重写5个函数跟是否继承MiddlrewareMixin无关
2. process_template_response 和 process_response必须要返回一个response值

#### 返回HTML模板
如果在视图中返回一个html模板,流程会是:
```text
md1.process_request
md2.process_request
md1.process_view
md2.process_view
IndexApiView.get
md2.process_response
md1.process_response
```
没有执行process_exception 和 process_template_response函数.

#### 返回HttpResponse对象
当返回HttpResponse对象时打印的LOG:
```text
md1.process_request
md2.process_request
md1.process_view
md2.process_view
IndexApiView.get
md2.process_template_response
md1.process_template_response
md2.process_response
md1.process_response
```
相比上一条,多执行了process_template_response函数

#### 抛出异常
如果在视图中抛出异常:
```python
md1.process_request
md2.process_request
md1.process_view
md2.process_view
IndexApiView.get
md2.process_exception
md1.process_exception
Exception: 抛出异常
md2.process_response
md1.process_response
```

#### 总结
##### process_request
1.
process_request有一个参数，就是request，这个request和视图函数中的request是一样的。
2.
它的返回值可以是None也可以是HttpResponse对象。返回值是None的话，按正常流程继续走，交给下一个中间件处理，如果是HttpResponse对象，Django将不执行视图函数，而将相应对象返回给浏览器.
3. 中间件的process_request方法是在执行视图函数之前执行的。
4.
当配置多个中间件时，会按照MIDDLEWARE中的注册顺序，也就是列表的索引值，从前到后依次执行的。
不同中间件之间传递的request都是同一个对象
5. 返回None,不错任何处理直接进行下一步
6. 返回值
2.返回响应对象，直接跳出（后续中间件的process_request、不执行urls.py和views.py）返回响应

##### process_view
该方法有四个参数
request是HttpRequest对象。
view_func是Django即将使用的视图函数。
（它是实际的函数对象，而不是函数的名称作为字符串。）
view_args是将传递给视图的位置参数的列表.
view_kwargs是将传递给视图的关键字参数的字典。
view_args和view_kwargs都不包含第一个视图参数（request）。
Django会在调用视图函数之前调用process_view方法。
它应该返回None或一个HttpResponse对象。
如果返回None，Django将继续处理这个请求，执行任何其他中间件的process_view方法，然后在执行相应的视图。
如果它返回一个HttpResponse对象，Django不会调用适当的视图函数。
它将执行中间件的process_response方法并将应用到该HttpResponse并返回结果
process_view方法是在process_request之后，视图函数之前执行的，执行顺序按照MIDDLEWARE中的注册顺序从前到后顺序执行的

1. 在urls.py之后在执行真正的视图函数之前
2. 按照在列表中注册的顺序依次执行
3. 返回None,放行
4. 返回响应对象，就直接跳出，倒序依次执行所有中间件的process_response方法

##### process_response
它有两个参数，一个是request，一个是response，request就是上述例子中一样的对象，response是视图函数返回的HttpResponse对象。
该方法的返回值也必须是HttpResponse对象。
process_response方法是在视图函数之后执行的，并且顺序是逆向的.
多个中间件中的process_response方法是按照MIDDLEWARE中的注册顺序倒序执行的，也就是说第一个中间件的process_request方法首先执行，
而它的process_response方法最后执行，最后一个中间件的process_request方法最后一个执行，它的process_response方法是最先执行。
1. 在views.py返回响应对象之后执行
2. 执行的顺序按照在列表中注册的倒序依次执行
3. 返回值必须要有返回值，返回要是响应对象

##### process_exception
该方法两个参数:一个HttpRequest对象，另一个exception是视图函数异常产生的Exception对象。
这个方法只有在视图函数中出现异常了才执行，它返回的值可以是一个None也可以是一个HttpResponse对象。如果是HttpResponse对象，Django将调用模板和中间件中的process_response方法，并返回给浏览器，否则将默认处理异常。如果返回一个None，则交给下一个中间件的process_exception方法来处理异常。它的执行顺序也是按照中间件注册顺序的倒序执行。

##### process_template_response
它的参数，一个HttpRequest对象，response是TemplateResponse对象（由视图函数或者中间件产生）。
process_template_response是在视图函数执行完成后立即执行，但是它有一个前提条件，那就是视图函数返回的对象有一个render()方法
视图函数执行完之后，立即执行了中间件的process_template_response方法，顺序是倒序，先执行MD1的，在执行MD2的，接着执行了视图函数返回的HttpResponse对象的render方法，返回了一个新的HttpResponse对象，接着执行中间件的process_response方法。


### 总结
1. request请求经过WSGI后，先进入中间件，依然开始先走process_request函数，然后走路由关系映射后，这里注意并没有直接进入视图函数，而是从头开始执行process_view()函数；然后再去执行与urls.py匹配的视图函数；
2. 如果视图函数没有报错，那就直接挨个反过来从最后一个中间件开始，依次将返回的实例对象(也就是我们在视图函数中写的
return HttpResponse()等等)传递给每个中间件的process_response函数；最后再交给客户端浏览器；
3. 如果执行视图函数出错，那就反过来从最后一个中间件开始，将错误信息传递给每个中间件的process_exception()函数，走完所有后，然后最终再走procss_response后，最终再交给客户端浏览器注意：视图函数的错误是由process_exception()函数处理的，从最后一个中间件开始，依次逐级提交捕捉到的异常然后最终交给procss_response()函数，将最终的错误信息交给客户端浏览器。
4. 过程
![过程](https://qxinhai.oss-cn-shenzhen.aliyuncs.com/pic/55-2.jpg)


### 参考
```html
https://zhuanlan.zhihu.com/p/39275116
https://blog.csdn.net/mbl114/article/details/78220606
https://blog.csdn.net/lm_is_dc/article/details/80527298
https://code.ziqiangxuetang.com/django/django-middleware.html
```