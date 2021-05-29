title: Django-rest-framework --- 限流
author: _Tao
tags:
  - django
  - django-rest-framework
categories:
  - Python
date: 2020-05-23 19:13:00
---
### 简介
限流就是限制访问,就是通常一个用户在多次请求一个页面，或者点击一个链接的时候，前几次点击是没有问题的，但是一旦连续几次之后，就会出现访问受限，离下一次访问还有50秒等的字样，在django
rest framework 中有一个专门的组件来做限制访问.

<!-- more -->

### 代码
#### 自定义Throttle类
```python
class AllUserThrottle(throttling.SimpleRateThrottle):
    # 使用的缓存方式
    cache = cache
    # 必填
    scope = 'all_user'
 
    def get_cache_key(self, request, view):
        if request.user.is_authenticated:
            # 如果是登录用户,则使用scope=user
            AllUserThrottle.scope = 'user'
            ident = request.user.username
        else:
            # 如果是未登录用户,则使用scope='anonymous'
            AllUserThrottle.scope = 'anonymous'
            ident = self.get_ident(request)
 
        return self.cache_format % {
            'scope': self.scope,
            'ident': ident
        }
```

#### 配置
```python
REST_FRAMEWORK = {
    'DEFAULT_THROTTLE_RATES': {
        'anonymous': '5/m',
        'user': '10/m',
        'all_user': '15/m',
    }
}
```

#### 视图
```python
class ThrottleApiView(generics.ListAPIView):
    queryset = UserInfo.objects.all()
    # 使用限流的方式
    throttle_classes = [AllUserThrottle]
 
    def list(self, request, *args, **kwargs):
        userinfo = [qs.user.username for qs in self.get_queryset()]
 
        return JsonResponse(json.dumps(userinfo), safe=False)
```

#### 小结
当用户登录状态访问ThrottleApiView时,只能每分钟访问10次,如果是未登录状态访问,只能每分钟访问5次.
上面的代码写法没太大必要,可以仿照drf的AnonRateThrottle和UserRateThrottle,根据不同的状态来调用不同的限流类,而不是用一个限流类根据不同的状态来使用不同的scope,太低级了.

### AnonRateThrottle 和 UserRateThrottle
对于根据用户是否登录来限制访问次数,drf提供了AnonRateThrottle 和
UserRateThrottle两个类,只需要在配置文件中配置下即可限制登录用户访问10次/分钟,未登录用户限制5次/分钟了.

```
REST_FRAMEWORK = {
    'DEFAULT_THROTTLE_CLASSES':[
      'rest_framework.throttling.AnonRateThrottle',
      'rest_framework.throttling.UserRateThrottle',
    ],
    'DEFAULT_THROTTLE_RATES': {
        'anon': '5/m',
        'user': '10/m',
    }
}
```
当然,不推荐上述写法,最好把限流类写入视图类中,不要全局定义.

### 源代码

#### initial
rest_framework.views.APIView:
会在初始化时,检查限流
```python
def initial(self, request, *args, **kwargs):
    ...
    ...
    self.check_throttles(request)
```
#### check_throttles
rest_framework.views.APIView:
会获取自定义的throttle_classes列表(如果没有则使用默认的),调用allow_request函数判断是否被限流,如果限流了就
抛出异常,并提示还需要多久才能再次访问
```python
def check_throttles(self, request):
    """
    Check if request should be throttled.
    Raises an appropriate exception if the request is throttled.
    """
    for throttle in self.get_throttles():
        # 当返回None或者False会抛出异常
        # 返回为True,表示可以正常访问
        if not throttle.allow_request(request, self):
            self.throttled(request, throttle.wait())
 
# 获取throttle_class列表数据
def get_throttles(self):
    """
    Instantiates and returns the list of throttles that this view uses.
    """
    return [throttle() for throttle in self.throttle_classes]
# 默认值
throttle_classes = api_settings.DEFAULT_THROTTLE_CLASSES
```

#### SimpleRateThrottle
rest_framework.throttling:
一般我们自定义限流类都会继承该类,并且重写scope属性,以及get_cache_key()函数.
```python
class SimpleRateThrottle(BaseThrottle):
    # 使用哪种缓存
    cache = default_cache
    # 时间戳
    timer = time.time
    # 存入缓存时的key值
    cache_format = 'throttle_%(scope)s_%(ident)s'
    # 自定义时必须重写
    scope = None
    # 频率设置 { ‘user’: ‘10/m’ }
    THROTTLE_RATES = api_settings.DEFAULT_THROTTLE_RATES
 
    def __init__(self):
        if not getattr(self, 'rate', None):
            self.rate = self.get_rate()
        # 配置文件中一定访问时间内的访问次数
        self.num_requests, self.duration = self.parse_rate(self.rate)
 
    # 自定义时必须重写,放回某限流下的唯一值
    def get_cache_key(self, request, view):
        raise NotImplementedError('.get_cache_key() must be overridden')
 
    def get_rate(self):
        # 没有scope时抛出异常
        if not getattr(self, 'scope', None):
            msg = ("You must set either `.scope` or `.rate` for '%s'
throttle" %
                   self.__class__.__name__)
            raise ImproperlyConfigured(msg)
 
        try:
            # 返回频率,类似 ‘10/m’之类
            return self.THROTTLE_RATES[self.scope]
        except KeyError:
            msg = "No default throttle rate set for '%s' scope" %
self.scope
            raise ImproperlyConfigured(msg)
 
    # 返回频率元祖
    def parse_rate(self, rate):
        if rate is None:
            return (None, None)
        num, period = rate.split('/')
        num_requests = int(num)
        duration = {'s': 1, 'm': 60, 'h': 3600, 'd': 86400}[period[0]]
        return (num_requests, duration)
 
    def allow_request(self, request, view):
        # 是否限制访问频率
        # 当限制的频率为空时,不会抛出异常
        if self.rate is None:
            return True
 
        # 获取用户的唯一标志
        self.key = self.get_cache_key(request, view)
        if self.key is None:
            return True
 
        # 获取用户的历史访问次数
        self.history = self.cache.get(self.key, [])
        # 当前时间
        self.now = self.timer()
 
        #
如果之前已经访问过该链接,但距离当前时间最长的一次访问超过了duration,
        # 就将其删除
        # 循环判断
        while self.history and self.history[-1] <= self.now -
self.duration:
            self.history.pop()
        #
删除"过期"时间后,剩下的访问次数仍然大于等于限制次数,那么就限制访问
        if len(self.history) >= self.num_requests:
            return self.throttle_failure()
        # 正常访问
        return self.throttle_success()
 
    # 可以正常访问
    # 将访问时间插入列表首位
    # 更新缓存信息
    def throttle_success(self):
        self.history.insert(0, self.now)
        self.cache.set(self.key, self.history, self.duration)
        return True
 
    # 限制访问
    def throttle_failure(self):
        return False
 
    # 不可访问时的等待时间
    def wait(self):
        # 如果有访问记录,就返回限制时间-最新一次访问时间
        # 否则返回总的限制时间
        if self.history:
            remaining_duration = self.duration - (self.now -
self.history[-1])
        else:
            remaining_duration = self.duration
 
        # TODO 没看懂
        available_requests = self.num_requests - len(self.history) + 1
        if available_requests <= 0:
            return None
 
        return remaining_duration / float(available_requests)
```

#### get_ident
rest_framework.BaseThrottle:
如果用户没有返回唯一的值,那么可以调用该函数,用ip地址做唯一值
```python
def get_ident(self, request):
    xff = request.META.get('HTTP_X_FORWARDED_FOR')
    remote_addr = request.META.get('REMOTE_ADDR')
    num_proxies = api_settings.NUM_PROXIES
 
    if num_proxies is not None:
        if num_proxies == 0 or xff is None:
            return remote_addr
        addrs = xff.split(',')
        client_addr = addrs[-min(num_proxies, len(addrs))]
        return client_addr.strip()
 
    return ''.join(xff.split()) if xff else remote_addr
```

#### drf定义限流类介绍
rest_framework.throttling:
```python
class UserRateThrottle(SimpleRateThrottle):
    scope = 'user'
    def get_cache_key(self, request, view):
        # 如果登录了,则返回pk
        # 否则返回ip
        if request.user.is_authenticated:
            ident = request.user.pk
        else:
            ident = self.get_ident(request)
 
        # 需要拼接字符串
        return self.cache_format % {
            'scope': self.scope,
            'ident': ident
        }
 
 
class AnonRateThrottle(SimpleRateThrottle):
    scope = 'anon'
    def get_cache_key(self, request, view):
        #
为了避免与user的冲突,如果在设置anon时,如果已经登录了,那么就返回None,不记录次数
        if request.user.is_authenticated:
            return None  # Only throttle unauthenticated requests.
 
        # 拼接字符串
        return self.cache_format % {
            'scope': self.scope,
            'ident': self.get_ident(request)
        }
```

#### 自定义throttle异常
在APIView类下有个throttled()函数,重写该函数就可实现自定义异常
rest_framework.views.APIView:
```python
def throttled(self, request, wait):
    """
    If request is throttled, determine what kind of exception to raise.
    """
    raise exceptions.Throttled(wait)
 
# rest_framework.exceptions:
class Throttled(APIException):
    status_code = status.HTTP_429_TOO_MANY_REQUESTS
    default_detail = _('Request was throttled.')
    extra_detail_singular = 'Expected available in {wait} second.'
    extra_detail_plural = 'Expected available in {wait} seconds.'
    default_code = 'throttled'
 
    def __init__(self, wait=None, detail=None, code=None):
        if detail is None:
            detail = force_text(self.default_detail)
        if wait is not None:
            wait = math.ceil(wait)
            detail = ' '.join((
                detail,
force_text(ungettext(self.extra_detail_singular.format(wait=wait),
self.extra_detail_plural.format(wait=wait),
                                     wait))))
        self.wait = wait
        super(Throttled, self).__init__(detail, code)
```
例如:
```python
from rest_framework import exceptions
def throttled(self, request, wait):
 
    class MyThrottled(exceptions.Throttled):
        default_detail = '请求被限制.'
        extra_detail_singular = 'Expected available in {wait} second.'
        extra_detail_plural = '还需要再等待{wait}'
 
    raise MyThrottled(wait)
```

### 参考
```text
https://www.cnblogs.com/cjaaron/p/10443725.html
https://www.cnblogs.com/eric_yi/p/8424424.html
https://blog.csdn.net/qq_42487752/article/details/85328307
https://www.cnblogs.com/supery007/p/8423769.html
https://www.cnblogs.com/welan/p/10138615.html
```