title: django --- session源码分析
author: _Tao
tags:
  - django
  - 源码
categories:
  - python
date: 2020-05-23 19:29:00
---
### cookie和seesion的区别
简单来讲就是都是key-value形式的数据，不同的是，cookie存储在客户端，session存储在服务端。具体的解释可以上网搜，不做叙述。

<!-- more -->

### Session模型
在`settings.py`中的`INSTALLED_APPS`增加`django.contrib.sessions`。

数据模型:
```python
class AbstractBaseSession(models.Model):
    session_key = models.CharField(_('session key'), max_length=40, primary_key=True)
    session_data = models.TextField(_('session data'))
    expire_date = models.DateTimeField(_('expire date'), db_index=True)

    objects = BaseSessionManager()

    class Meta:
        abstract = True
        verbose_name = _('session')
        verbose_name_plural = _('sessions')
```
```python
class Session(AbstractBaseSession):

    class Meta(AbstractBaseSession.Meta):
        db_table = 'django_session'
```
综上，那么在执行完`python manage.py migrate`指令后，将在数据库中生成`django_session`数据表，表中字段只有三个<br/>
`session_key`，`session_data`，`expire_date`。<br/>
`session_key`,会在cookie中存储一个唯一性的`session_id`，在客户端向服务端发送请求中，会从cookie中获取到这个id值，从数据表中获取对应的数据。<br/>
`session_data`,存储加密后的数据，例如
```
request.session['hello'] = 'world'
request.session.save()
```
存储的后数据就是`NjdjMmY3ZGRlMDNmNTRlM2FlMTNjNmMzODMzYzgyNTgxMjE5NDVkNjp7ImhlbGxvIjoid29ybGQifQ`，根据`secret_key`不同，存储后的数据也肯定会不同。<br/>
`expire_date`，session的有效期。
举例:<br/>
在cookie中有数据, key="session_id", value="r7bwqr8ba5pypolyq4e502zjxaxcl9v5", 那么此value就是`django_session`表的`session_key`数据。


### session存储方式
从django的代码路径`django.contrib.sessions`可以获取到具体分类，具体使用哪种可以在`settings.py`中配置`SESSION_ENGINE`属性，例如
```python
SESSION_ENGINE = 'django.contrib.sessions.backends.signed_cookies'
```
#### django.contrib.sessions.file
将session数据存放在文件中，一般文件都是放在/tmp下，例如windows平台就是在`C:\Users\Administrator\AppData\Local\Temp`路径下。<br/>
不推荐。

#### django.contrib.sessions.cache
将session数据放在缓存中。<br/>
使用redis做缓存配置如下:
```python
CACHES = {
    "default": {
        "BACKEND": "django_redis.cache.RedisCache",
        "LOCATION": "redis://127.0.0.1:6379/1",
        "OPTIONS": {
            "CLIENT_CLASS": "django_redis.client.DefaultClient",
        }
    }
}
SESSION_ENGINE = "django.contrib.sessions.backends.cache"
SESSION_CACHE_ALIAS = "default"
```

#### django.contrib.sessions.db
将session放在数据库中，这个是默认方式。
```python
# The module to store session data
SESSION_ENGINE = 'django.contrib.sessions.backends.db'
```
#### django.contrib.sessions.cache_db
混合存储，优先从缓存中获取session，如果没有再从数据库中获取，结合了上面两种。

#### django.contrib.sessions.signed_cookies
将session数据加密后放在cookie中。<br/>
不推荐。

### 源码介绍
看了下代码，发现session的代码比我想象中的简单太多，就稍微拿数据库存储方式说下。

#### SessionBase
session的所有存储方式的类都叫做`SessionStore`，而所有的类都继承自`SessionBase`。<br/>
在使用session时完成就是按照dict使用的，看了下它的代码，虽然说并没有继承自dict或跟dict相关的类，但其实一看就跟dict的用法一致。<br/>
路径：`django.contrib.sessions.backends.base.SessionBase`

##### 初始化数据
```python
def __init__(self, session_key=None):
    # 客户端的session_id的值
    self._session_key = session_key
    # 是否访问
    self.accessed = False
    # 是否修改
    self.modified = False
    # 序列化方式，默认是json
    self.serializer = import_string(settings.SESSION_SERIALIZER)
```

##### 其他常用方法
具体的可以查看源码
```python
def __contains__(self, key):
    """
    判断session是否存在
    """
    return key in self._session

def __getitem__(self, key):
    """
    获取session数据
    """
    return self._session[key]

def __setitem__(self, key, value):
    """
    设置数据
    """
    self._session[key] = value
    # 因为设置了数据，那么就是说已经修改了
    self.modified = True

def __delitem__(self, key):
    """
    删除数据
    """
    del self._session[key]
    self.modified = True

def get(self, key, default=None):
    return self._session.get(key, default)

def pop(self, key, default=__not_given):
    self.modified = self.modified or key in self._session
    # 在pop数据时，是否有存入默认值
    args = () if default is self.__not_given else (default,)
    return self._session.pop(key, *args)

def setdefault(self, key, value):
    if key in self._session:
        return self._session[key]
    else:
        self.modified = True
        self._session[key] = value
        return value
```

##### 继承时需要实现的方法
```python
def exists(self, session_key):
    """
    Returns True if the given session_key already exists.
    """
    raise NotImplementedError('subclasses of SessionBase must provide an exists() method')

def create(self):
    """
    Creates a new session instance. Guaranteed to create a new object with
    a unique key and will have saved the result once (with empty data)
    before the method returns.
    """
    raise NotImplementedError('subclasses of SessionBase must provide a create() method')

def save(self, must_create=False):
    """
    Saves the session data. If 'must_create' is True, a new session object
    is created (otherwise a CreateError exception is raised). Otherwise,
    save() only updates an existing object and does not create one
    (an UpdateError is raised).
    """
    raise NotImplementedError('subclasses of SessionBase must provide a save() method')

def delete(self, session_key=None):
    """
    Deletes the session data under this key. If the key is None, the
    current session key value is used.
    """
    raise NotImplementedError('subclasses of SessionBase must provide a delete() method')

def load(self):
    """
    Loads the session data and returns a dictionary.
    """
    raise NotImplementedError('subclasses of SessionBase must provide a load() method')

@classmethod
def clear_expired(cls):
    """
    Remove expired sessions from the session store.

    If this operation isn't possible on a given backend, it should raise
    NotImplementedError. If it isn't necessary, because the backend has
    a built-in expiration mechanism, it should be a no-op.
    """
    raise NotImplementedError('This backend does not support clear_expired().')
```

##### _get_session方法
在代码中，获取或设置数据时都是调用的`self._session`,从代码看
```python
def _get_session(self, no_load=False):
    """
    Lazily loads session from storage (unless "no_load" is True, when only
    an empty dict is stored) and stores it in the current instance.
    """
    self.accessed = True
    try:
        return self._session_cache
    except AttributeError:
        if self.session_key is None or no_load:
            self._session_cache = {}
        else:
            self._session_cache = self.load()
    return self._session_cache

_session = property(_get_session)
```
`_session`是通过调用`self.load()`获取到的，也就是具体逻辑要子类实现了，根据存储的方式不同实现的代码也就不同。

#### SessionStore，db方式

##### 获取db方式数据表模型
也就是最开始提到的`Session`模型了。
```python
@classmethod
def get_model_class(cls):
    from django.contrib.sessions.models import Session
    return Session

@cached_property
def model(self):
    return self.get_model_class()
```

##### load
实现父类的load方法。<br/>
其实就是一般的orm使用方法，如果存在返回数据的Session对象，不存在则返回空dict，额外做了一个有效期的判断。
```python
def load(self):
    try:
        s = self.model.objects.get(
            session_key=self.session_key,
            expire_date__gt=timezone.now()
        )
        return self.decode(s.session_data)
    except (self.model.DoesNotExist, SuspiciousOperation) as e:
        self._session_key = None
        return {}
```

##### create
创建Session对象。
_get_new_session_key()方法会生成一个32位的随机字符串，生成后就立即保存了，避免key值出现重复。<br/>
这儿稍微提了下，因为在工作中好像遇到过这类问题。
```python
def create(self):
    while True:
        self._session_key = self._get_new_session_key()
        try:
            # Save immediately to ensure we have a unique entry in the database.
            self.save(must_create=True)
        except CreateError:
            # Key wasn't unique. Try again.
            continue
        self.modified = True
        return
```

#### 中间件SessionMiddleware
```python
class SessionMiddleware(MiddlewareMixin):
    def __init__(self, get_response=None):
        self.get_response = get_response
        # session的存储方式是可配置的，那么根据settings.py中的配置获取不同的SessionStore类
        # import_module根据路径（字符串）获取模块对象
        engine = import_module(settings.SESSION_ENGINE)
        self.SessionStore = engine.SessionStore

    def process_request(self, request):
        # 存放在cookie中的django_session表的session_key对应的key值，默认是sessionid
        session_key = request.COOKIES.get(settings.SESSION_COOKIE_NAME)
        # 获取session对象
        request.session = self.SessionStore(session_key)

    def process_response(self, request, response):
        try:
            # 是否已访问
            accessed = request.session.accessed
            # 是否已修改
            modified = request.session.modified
            # 是否为空
            empty = request.session.is_empty()
        except AttributeError:
            pass
        else:
            # First check if we need to delete this cookie.
            # The session should be deleted only if the session is entirely empty
            if settings.SESSION_COOKIE_NAME in request.COOKIES and empty:
                response.delete_cookie(
                    settings.SESSION_COOKIE_NAME,
                    path=settings.SESSION_COOKIE_PATH,
                    domain=settings.SESSION_COOKIE_DOMAIN,
                )
            else:
                if accessed:
                    patch_vary_headers(response, ('Cookie',))
                if (modified or settings.SESSION_SAVE_EVERY_REQUEST) and not empty:
                    # 确定session的有效期
                    if request.session.get_expire_at_browser_close():
                        max_age = None
                        expires = None
                    else:
                        max_age = request.session.get_expiry_age()
                        expires_time = time.time() + max_age
                        expires = cookie_date(expires_time)
                    if response.status_code != 500:
                        try:
                            # 服务端保存
                            request.session.save()
                        except UpdateError:
                            return redirect(request.path)
                        # 客户端保存
                        response.set_cookie(
                            settings.SESSION_COOKIE_NAME,
                            request.session.session_key, max_age=max_age,
                            expires=expires, domain=settings.SESSION_COOKIE_DOMAIN,
                            path=settings.SESSION_COOKIE_PATH,
                            secure=settings.SESSION_COOKIE_SECURE or None,
                            httponly=settings.SESSION_COOKIE_HTTPONLY or None,
                        )
        return response
```

### 总结
之前一直想把这部分源码过一遍，但拖延症太严重一直放着了。现在花点时间看了下源码，比想象中的简单好多，一下子都不知道笔记该怎么写了，就随便复制了点看起来重要的代码添加点注释了。<br/>
session模块主要的代码都在`django.contrib.session.backends`路径下，代码很简单，配合django的注释很容易读懂，就这样了。