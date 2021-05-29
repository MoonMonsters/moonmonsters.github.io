title: Django --- 请求到响应源码分析
author: _Tao
tags:
  - django
  - 源码
categories:
  - Python
date: 2020-05-23 19:12:00
---
web应用或者网站本质上都是围绕着请求-响应的方式来运作的。当你通过浏览器访问网站时，浏览器会向web服务器发送请求。当web服务器收到请求后，服务器会对请求进行相应的处理，然后返回相应的响应给浏览器，最后浏览器呈现给你。

### 入口
这是web服务器转发请求到django应用的地方，也是返回响应的地方。

路径: `xxx.wsgi.py`
```python
import os

from django.core.wsgi import get_wsgi_application
# 设置django要使用的配置文件
os.environ.setdefault('DJANGO_SETTINGS_MODULE', 'JwtDemo.settings')

application = get_wsgi_application()
```

<!-- more -->

### WSGIHandler

初始化请求，并返回响应数据。

```python
class WSGIHandler(base.BaseHandler):
    request_class = WSGIRequest

    def __init__(self, *args, **kwargs):
        super().__init__(*args, **kwargs)
        # 加载django应用中配置的中间件，该方法是父类BaseHandler中的
        self.load_middleware()

    def __call__(self, environ, start_response):
        set_script_prefix(get_script_name(environ))
        signals.request_started.send(sender=self.__class__, environ=environ)
        # 初始化request请求
        request = self.request_class(environ)
        # 开始处理请求，并生成响应
        response = self.get_response(request)

        response._handler_class = self.__class__

        # 状态码，状态描述
        status = '%d %s' % (response.status_code, response.reason_phrase)
        # 响应头信息
        response_headers = list(response.items())
        for c in response.cookies.values():
            response_headers.append(('Set-Cookie', c.output(header='')))
        start_response(status, response_headers)
        if getattr(response, 'file_to_stream', None) is not None and environ.get('wsgi.file_wrapper'):
            response = environ['wsgi.file_wrapper'](response.file_to_stream)
        return response
```

### 加载中间件

#### load_middleware
路径:`django.core.handlers.base.BaseHandler.load_middleware`
WSGIHandler中调用的load_middleware函数是父类BaseHandler中的，在这个函数中，会按顺序加载所有配置的中间件。
主循环会对所有的中间件逆序遍历，而将中间件函数加入列表的顺序也跟实际的执行顺序一致。
`process_view`会按照中间件的先后顺序执行，所以采用了insert(0,xx),而`process_template_response`和`process_exception`则采用了append操作。
这个看着有点绕..
以下面的中间件配置为例:
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
因为采用了逆序遍历，那么执行顺序就是`XFrameOptionsMiddleware -> MessageMiddleware -> ... ->SecurityMiddleware`，而`process_view`采用了insert(0,xx)的方法，所以`_view_middleware`中`process_view`函数的顺序就是`SecurityMiddleware.process_view -> ... -> SecurityMiddleware.process_view`了。

```python
    def load_middleware(self):
        # 加载中间件

        # 中间件中的process_view函数列表
        self._view_middleware = []
        # 中间件中的process_template_response函数列表
        self._template_response_middleware = []
        # 中间件中的process_exception函数列表
        self._exception_middleware = []

        # 这是一个装饰器，装饰器内返回了response，只不过包裹了exception数据
        handler = convert_exception_to_response(self._get_response)
        # middleware_path是字符串
        for middleware_path in reversed(settings.MIDDLEWARE):
            # 转换成具体的类
            middleware = import_string(middleware_path)
            try:
                mw_instance = middleware(handler)
            except MiddlewareNotUsed as exc:
                if settings.DEBUG:
                    if str(exc):
                        logger.debug('MiddlewareNotUsed(%r): %s', middleware_path, exc)
                    else:
                        logger.debug('MiddlewareNotUsed: %r', middleware_path)
                continue

            if mw_instance is None:
                raise ImproperlyConfigured(
                    'Middleware factory %s returned None.' % middleware_path
                )

            # 对不同的中间件函数，使用不同的插入顺序，随他们的处理顺序
            # process_view正序插入
            if hasattr(mw_instance, 'process_view'):
                self._view_middleware.insert(0, mw_instance.process_view)
            # process_template_response逆序插入
            if hasattr(mw_instance, 'process_template_response'):
                self._template_response_middleware.append(mw_instance.process_template_response)
            # process_exception逆序插入
            if hasattr(mw_instance, 'process_exception'):
                self._exception_middleware.append(mw_instance.process_exception)

            # 重新处理
            handler = convert_exception_to_response(mw_instance)

        # 是一个修饰器函数，最内层是_get_response，外层是中间件
        self._middleware_chain = handler
```

#### MiddlewareMixin

使用了mixin模式，当自定义中间件时，需要自己实现`process_request`和`process_response`两个函数。
```python
class MiddlewareMixin:
    def __init__(self, get_response=None):
        self.get_response = get_response
        super().__init__()

    def __call__(self, request):
        response = None
        # 处理proecess_request函数
        if hasattr(self, 'process_request'):
            response = self.process_request(request)
        response = response or self.get_response(request)
        # 处理process_response函数
        if hasattr(self, 'process_response'):
            response = self.process_response(request, response)
        return response
```

#### get_response
路径:`django.core.handlers.base.get_response`

```python
    def get_response(self, request):
        """Return an HttpResponse object for the given HttpRequest."""
        # 设置开始位置的url
        set_urlconf(settings.ROOT_URLCONF)

        # 包装了self._get_response的装饰器
        response = self._middleware_chain(request)

        response._closable_objects.append(request)

        # If the exception handler returns a TemplateResponse that has not
        # been rendered, force it to be rendered.
        if not getattr(response, 'is_rendered', True) and callable(getattr(response, 'render', None)):
            response = response.render()

        if response.status_code >= 400:
            log_response(
                '%s: %s', response.reason_phrase, request.path,
                response=response,
                request=request,
            )

        return response
```

#### _get_response
路径: `django.core.handlers.base._get_response`
这个函数是真正的调用视图函数或者视图类的地方
```python
    def _get_response(self, request):
        """
        Resolve and call the view, then apply view, exception, and
        template_response middleware. This method is everything that happens
        inside the request/response middleware.
        实际上调用视图函数的地方
        """
        response = None

        if hasattr(request, 'urlconf'):
            urlconf = request.urlconf
            set_urlconf(urlconf)
            resolver = get_resolver(urlconf)
        else:
            resolver = get_resolver()

        # 通过url操作，获取实际的执行函数
        resolver_match = resolver.resolve(request.path_info)
        callback, callback_args, callback_kwargs = resolver_match
        request.resolver_match = resolver_match

        # 执行proecess_view中间件函数
        for middleware_method in self._view_middleware:
            # 调用process_view函数
            response = middleware_method(request, callback, callback_args, callback_kwargs)
            # 如果某个中间件的process_view返回了值，那么就停止继续执行
            if response:
                break
        
        if response is None:
            # callback是实际上要执行的视图函数或者视图类
            wrapped_callback = self.make_view_atomic(callback)
            try:
                # 执行实际函数，获取返回的response
                response = wrapped_callback(request, *callback_args, **callback_kwargs)
            except Exception as e:
                response = self.process_exception_by_middleware(e, request)

        # Complain if the view returned None (a common error).
        if response is None:
            if isinstance(callback, types.FunctionType):    # FBV
                view_name = callback.__name__
            else:                                           # CBV
                view_name = callback.__class__.__name__ + '.__call__'

            raise ValueError(
                "The view %s.%s didn't return an HttpResponse object. It "
                "returned None instead." % (callback.__module__, view_name)
            )

        # If the response supports deferred rendering, apply template
        # response middleware and then render the response
        # 判断是否需要执行中间件中的process_template_response函数
        elif hasattr(response, 'render') and callable(response.render):
            # 遍历执行process_template_response函数
            for middleware_method in self._template_response_middleware:
                response = middleware_method(request, response)
                # 如果某个process_template_response函数返回了None，则抛出异常
                # 自定义process_template_response函数时，需要返回response
                if response is None:
                    raise ValueError(
                        "%s.process_template_response didn't return an "
                        "HttpResponse object. It returned None instead."
                        % (middleware_method.__self__.__class__.__name__)
                    )

            try:
                # 调用HttpResponse的render()函数，获取真正的返回值
                response = response.render()
            except Exception as e:
                response = self.process_exception_by_middleware(e, request)

        return response
```


#### HttpResponse.render
路径:`django.template.response.py`
```python
    def render(self):
        """Render (thereby finalizing) the content of the response.

        If the content has already been rendered, this is a no-op.

        Return the baked response instance.
        """
        retval = self
        if not self._is_rendered:
            self.content = self.rendered_content
            for post_callback in self._post_render_callbacks:
                newretval = post_callback(retval)
                if newretval is not None:
                    retval = newretval
        return retval
```

### 执行流程总结
→ wsgi.py是一次请求的入口
→ get_wsgi_application()，返回WSGI调用对象
→ WSGIHandler()，接收请求\返回响应对象的类
→ load_middleware()，在创建WSGIHandler()对象时，会调用此函数，加载配置文件中的中间件，在一次服务器启动期间，只执行一次
→ 将_get_response函数做参数传入convert_exception_to_response函数，执行函数会返回handler，该handler实际上是一个包裹了exception和_get_response函数的装饰器
→ 将handler赋值给_middleware_chain
→ 中间件加载完毕
→ 一次请求到达django时，会调用django的__call__函数，在该函数中，会调用get_response（定义在父类BaseHandler中）函数
→ 在get_response函数中，会执行response = self._middleware_chain(request)这一行代码，从上面的步骤看，也就相当于执行了_get_response函数
→ 在_get_response函数中，会执行真正的视图函数(callback和wrapped_callback)，返回的结果response对象
→ _get_response执行完毕后，返回response，回到get_response函数中
→ get_response函数拿到_get_response返回的response对象继续执行之后的代码，最后将response对象返回
→ get_response函数结束后，回到WSGIHandler的__call__函数中，执行后续代码，返回response到前端


### 总结
通过上面代码分析，我们已经大致了解了Django请求响应的流程。大致如下
用户请求首先会到web服务器；
web服务器会把请求发到django.core.handlers.wsgi的BaseHandler；
生成request，response，view， exception，template_response中间件链表；
按中间件配置顺序应用request中间件来处理request，如果这中间生成response，则直接返回；
通过urlresolvers.resolve匹配请求的url来找到对应的view；
应用view中间件，如果有response，则直接返回；
调用对应的view，这个过程和和models进行交互，比如从数据库获取数据等，并渲染模板；
接着response中间件会被应用来处理repsonse；
这其中忽略了一些其他重要的步骤，比如异常中间件的调用。

用网上的一张图总结以上：
![](https://qxinhai.oss-cn-shenzhen.aliyuncs.com/pic/django-%E8%AF%B7%E6%B1%82%E5%93%8D%E5%BA%94.png)


### 参考
```text
https://www.jianshu.com/p/1ff05dfb3d0d
http://hongweipeng.com/index.php/archives/1369/#menu_index_1
http://www.php.cn/python-tutorials-416971.html
https://www.cnblogs.com/time-read/p/10650988.html
```