title: django-rest-framework --- JWTAuthentication
author: _Tao
tags:
  - django
  - django-rest-framework
categories:
  - python
date: 2020-05-23 19:31:00
---
### 介绍

  JWT 是一个开放标准(RFC 7519)，它定义了一种用于简洁，自包含的用于通信双方之间以 JSON 对象的形式安全传递信息的方法。JWT 可以使用 HMAC 算法或者是 RSA 的公钥密钥对进行签名。它具备两个特点：


- 简洁(Compact)
可以通过URL, POST 参数或者在 HTTP header 发送，因为数据量小，传输速度快


- 自包含(Self-contained)
负载中包含了所有用户所需要的信息，避免了多次查询数据库

<!-- more -->

#### JWT 组成
- Header 头部
头部包含了两部分，token 类型和采用的加密算法,它会使用 Base64 编码组成 JWT 结构的第一部分.
```python
{
  "alg": "HS256",
  "typ": "JWT"
}
```

- Payload 负载
这部分就是我们存放信息的地方了，你可以把用户 ID 等信息放在这里，JWT 规范里面对这部分有进行了比较详细的介绍，常用的由 iss（签发者），exp（过期时间），sub（面向的用户），aud（接收方），iat（签发时间）。
同样的，它会使用 Base64 编码组成 JWT 结构的第二部分
```python
{
    "iss": "lion1ou JWT",
    "iat": 1441593502,
    "exp": 1441594722,
    "aud": "qxinhai.cn",
    "sub": "qxinhai@yeah.net"
}
```
- Signature 签名
前面两部分都是使用 Base64 进行编码的，即前端可以解开知道里面的信息。Signature 需要使用编码后的 header 和 payload 以及我们提供的一个密钥，然后使用 header 中指定的签名算法（HS256）进行签名。签名的作用是保证 JWT 没有被篡改过。
三个部分通过.连接在一起就是我们的 JWT 了，它可能长这个样子，长度貌似和你的加密算法和私钥有关系。
`eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpZCI6IjU3ZmVmMTY0ZTU0YWY2NGZmYzUzZGJkNSIsInhzcmYiOiI0ZWE1YzUwOGE2NTY2ZTc2MjQwNTQzZjhmZWIwNmZkNDU3Nzc3YmUzOTU0OWM0MDE2NDM2YWZkYTY1ZDIzMzBlIiwiaWF0IjoxNDc2NDI3OTMzfQ.PA3QjeyZSUh7H0GfE0vJaKW4LjKJuC3dVLQiY4hii8s`
其实到这一步可能就有人会想了，HTTP 请求总会带上 token，这样这个 token 传来传去占用不必要的带宽啊。如果你这么想了，那你可以去了解下 HTTP2，HTTP2 对头部进行了压缩，相信也解决了这个问题。

- 签名的目的
最后一步签名的过程，实际上是对头部以及负载内容进行签名，防止内容被窜改。如果有人对头部以及负载的内容解码之后进行修改，再进行编码，最后加上之前的签名组合形成新的JWT的话，那么服务器端会判断出新的头部和负载形成的签名和JWT附带上的签名是不一样的。如果要对新的头部和负载进行签名，在不知道服务器加密时用的密钥的话，得出来的签名也是不一样的。

- 信息暴露
在这里大家一定会问一个问题：Base64是一种编码，是可逆的，那么我的信息不就被暴露了吗？
是的。所以，在JWT中，不应该在负载里面加入任何敏感的数据。在上面的例子中，我们传输的是用户的User ID。这个值实际上不是什么敏感内容，一般情况下被知道也是安全的。但是像密码这样的内容就不能被放在JWT中了。如果将用户的密码放在了JWT中，那么怀有恶意的第三方通过Base64解码就能很快地知道你的密码了。
因此JWT适合用于向Web应用传递一些非敏感信息。JWT还经常用于设计用户认证和授权系统，甚至实现Web应用的单点登录。

#### JWT 使用
1. 首先，前端通过Web表单将自己的用户名和密码发送到后端的接口。这一过程一般是一个HTTP POST请求。建议的方式是通过SSL加密的传输（https协议），从而避免敏感信息被嗅探。
2. 后端核对用户名和密码成功后，将用户的id等其他信息作为JWT Payload（负载），将其与头部分别进行Base64编码拼接后签名，形成一个JWT。形成的JWT就是一个形同lll.zzz.xxx的字符串。
3. 后端将JWT字符串作为登录成功的返回结果返回给前端。前端可以将返回的结果保存在localStorage或sessionStorage上，退出登录时前端删除保存的JWT即可。
4. 前端在每次请求时将JWT放入HTTP Header中的Authorization位。(解决XSS和XSRF问题)
5. 后端检查是否存在，如存在验证JWT的有效性。例如，检查签名是否正确；检查Token是否过期；检查Token的接收方是否是自己（可选）。
6. 验证通过后后端使用JWT中包含的用户信息进行其他逻辑操作，返回相应结果。


### 示例
#### 安装
｀pip install djangorestframework-jwt｀

#### 配置
在settings.py中添加以下app
```python
'rest_framework',
'rest_framework_jwt',
```

#### 视图
```python
from rest_framework.views import APIView
from rest_framework.response import Response
from rest_framework_jwt.authentication import JSONWebTokenAuthentication
from rest_framework_jwt.settings import api_settings
from rest_framework import status

from django.contrib.auth.models import User

# 将用户数据添加进载荷
jwt_payload_handler = api_settings.JWT_PAYLOAD_HANDLER
# 加密
jwt_encode_handler = api_settings.JWT_ENCODE_HANDLER


class LoginView(APIView):
    """
    登录功能
    """
    def post(self, request):
        username = request.data.get('username')
        password = request.data.get('password')
        user = User.objects.filter(username=username).first()
        if user and user.check_password(password):
            payload = jwt_payload_handler(user)
            token = jwt_encode_handler(payload)

            return Response({'token': token}, status=status.HTTP_200_OK)

        return Response({'message': '登录失败'}, status=status.HTTP_401_UNAUTHORIZED)


class BooksView(APIView):
    """
    获取测试数据
    """
    authentication_classes = [JSONWebTokenAuthentication, ]

    def get(self, request):
        data = [
            {'name': 'java', 'price': 57},
            {'name': 'python', 'price': 93}
        ]
        return Response({'books': data})
```

如果用户登录成功，将返回token值。
主要靠以下两个函数来生成token值：
```python
# 将用户数据添加进载荷
jwt_payload_handler = api_settings.JWT_PAYLOAD_HANDLER
# 加密
jwt_encode_handler = api_settings.JWT_ENCODE_HANDLER

payload = jwt_payload_handler(user)
token = jwt_encode_handler(payload)
```
下次访问时，需要使用token值来验证身份。
不管是前端访问，还是requests访问，都要严格按照以下格式要求:
```python
headers = {
    'Authorization': 'JWT ' + token
}
```
JWT和token之间有空格。

从源码看，继承顺序是这样的:
`JSONWebTokenAuthentication -> BaseJSONWebTokenAuthentication -> BaseAuthentication`，最终也是继承自`BaseAuthentication`，也同样有个跟`TokenAuthentication`和`BasicAuthentication`的问题，那就是没有对`如果不传入Authorization数据`这种情况抛异常处理。详细讲就是，一个视图类使用了`JSONWebTokenAuthentication`认证，如果你不传入`Authorization`数据，它不会抛异常，但如果你传入了这个数据，数据却错了，就会抛异常。
看了认证相关的源码，都是在`authenticate()`函数中返回了`None`之后，就没有做任何判断了。不太清楚官方的正确做法应该是什么，但在解决`不传入Authorization数据却能正常访问`问题时，会使用以下两种方式。

1.  重写dispath和handle_exception函数
在调用get或者post函数之前，都会先调用dispatch函数，在此时判断用户是否验证成功.
验证成功时,`request.user`会返回登录的User对象，否则是`AnonymousUser`, `request.auth`也会打返回token值。在此时可以通过这两个数据，判断是否需要继续执行接下来实际的视图函数。
```python
class BasicApiView(APIView):
    """
    并不一定要继承APIView，可以按照需求继承
    """

    def dispatch(self, request, *args, **kwargs):
        """
        重写dispatch函数，在post和get之前调用
        """
        response = super(BasicApiView, self).dispatch(request, *args, **kwargs)
        # 登录的用户
        # request.user
        # token值
        # request.auth
        # 如果用户认证失败
        if not request.user or isinstance(request.user, AnonymousUser):
            try:
                # 抛出异常
                raise exceptions.AuthenticationFailed('认证失败')
            except exceptions.AuthenticationFailed as exc:
                # 捕获处理后的异常
                response = self.handle_exception(exc)
                # 一定要加上这一行，否则报错
                response = self.finalize_response(request, response, *args, **kwargs)

        return response

    def handle_exception(self, exc):
        """
        异常处理
        """
        # 如果是验证失败的异常，返回401
        if isinstance(exc, (exceptions.AuthenticationFailed, exceptions.NotAuthenticated)):
            return Response({'message': str(exc)}, status=status.HTTP_401_UNAUTHORIZED)

        # 其他的错误统一归为内部错误，如果有需要单独处理，放在之前用if判断处理
        return Response({'message': str(exc)}, status=status.HTTP_500_INTERNAL_SERVER_ERROR)


class BooksView(BasicApiView):
    """
    获取测试数据
    """
    authentication_classes = [JSONWebTokenAuthentication, ]

    def get(self, request):
        data = [
            {'name': 'java', 'price': 57},
            {'name': 'python', 'price': 93}
        ]
        return Response({'books': data})
```

2. 继承认证类，重写authenticate()函数

```python
class JWTAuthentication(JSONWebTokenAuthentication):
    def authenticate(self, request):
        # 获取返回值(user, token)
        user_auth_tuple = super().authenticate(request)

        # 如果返回值为空，则说明验证失败
        if not user_auth_tuple or not user_auth_tuple[0] or not user_auth_tuple[1]:
            raise exceptions.AuthenticationFailed('帐号或密码错误')

        # 否则继续往下执行
        return user_auth_tuple
```
然后在视图类中配置即可
```python
authentication_classes = [JWTAuthentication, ]
```

以上是最基本的用法，如果有定制要求，也不会差的太远。


### 源码
#### JSONWebTokenAuthentication
```python
    def get_jwt_value(self, request):
    	# 获取请求头中Authoriztion数据
        auth = get_authorization_header(request).split()
        # 获取标志位，如果没有自定义，就是 JWT
        auth_header_prefix = api_settings.JWT_AUTH_HEADER_PREFIX.lower()

        if not auth:
            if api_settings.JWT_AUTH_COOKIE:
                return request.COOKIES.get(api_settings.JWT_AUTH_COOKIE)
            return None

        if smart_text(auth[0].lower()) != auth_header_prefix:
            return None

        if len(auth) == 1:
            msg = _('Invalid Authorization header. No credentials provided.')
            raise exceptions.AuthenticationFailed(msg)
        elif len(auth) > 2:
            msg = _('Invalid Authorization header. Credentials string '
                    'should not contain spaces.')
            raise exceptions.AuthenticationFailed(msg)
	
		# 返回token数据
        return auth[1]
```

```python
    def authenticate(self, request):
		# token数据
        jwt_value = self.get_jwt_value(request)
        if jwt_value is None:
            return None
	
        try:
        	# 解析token值
        	# jwt_decode_handler = api_settings.JWT_DECODE_HANDLER
        	# 具体代码见源码分析
            payload = jwt_decode_handler(jwt_value)
        except jwt.ExpiredSignature:
            msg = _('Signature has expired.')
            raise exceptions.AuthenticationFailed(msg)
        except jwt.DecodeError:
            msg = _('Error decoding signature.')
            raise exceptions.AuthenticationFailed(msg)
        except jwt.InvalidTokenError:
            raise exceptions.AuthenticationFailed()

        user = self.authenticate_credentials(payload)
	
	# 认证成功后，返回登录的User对象和token数据
        return (user, jwt_value)
```

#### rest_framework_jwt.settings
路径:`rest_framework_jwt.settings.py`

我们一般都可以用CTRL + 左键查看函数\属性的具体信息，但我在下面使用下面两行代码时却发现无法点击，所以好奇就看了下相关的代码。
```python
# 将用户数据添加进载荷
jwt_payload_handler = api_settings.JWT_PAYLOAD_HANDLER
# 加密
jwt_encode_handler = api_settings.JWT_ENCODE_HANDLER
```

api_settings是APISettings的对象实例
```python
api_settings = APISettings(USER_SETTINGS, DEFAULTS, IMPORT_STRINGS)
```

settings实际上就是django运行时使用的配置文件
```python
from django.conf import settings
```

用户可以在配置文件中配置JWT_AUTH数据，所以在创建apt_settings对象时就从配置文件中将所有的用户自定义的数据读取出来，存放到USER_SETTINGS中，并传入ApiSettings中。
```python
USER_SETTINGS = getattr(settings, 'JWT_AUTH', None)
```

DEFAULTS中的数据是最齐全的，用户能自定义的数据都在该字典中
```python
DEFAULTS = {
    'JWT_ENCODE_HANDLER':
    'rest_framework_jwt.utils.jwt_encode_handler',

    'JWT_DECODE_HANDLER':
    'rest_framework_jwt.utils.jwt_decode_handler',

    'JWT_PAYLOAD_HANDLER':
    'rest_framework_jwt.utils.jwt_payload_handler',

    'JWT_PAYLOAD_GET_USER_ID_HANDLER':
    'rest_framework_jwt.utils.jwt_get_user_id_from_payload_handler',

    'JWT_PRIVATE_KEY':
    None,

    'JWT_PUBLIC_KEY':
    None,

    'JWT_PAYLOAD_GET_USERNAME_HANDLER':
    'rest_framework_jwt.utils.jwt_get_username_from_payload_handler',

    'JWT_RESPONSE_PAYLOAD_HANDLER':
    'rest_framework_jwt.utils.jwt_response_payload_handler',

    'JWT_SECRET_KEY': settings.SECRET_KEY,
    'JWT_GET_USER_SECRET_KEY': None,
    'JWT_ALGORITHM': 'HS256',
    'JWT_VERIFY': True,
    'JWT_VERIFY_EXPIRATION': True,
    'JWT_LEEWAY': 0,
    'JWT_EXPIRATION_DELTA': datetime.timedelta(seconds=300),
    'JWT_AUDIENCE': None,
    'JWT_ISSUER': None,

    'JWT_ALLOW_REFRESH': False,
    'JWT_REFRESH_EXPIRATION_DELTA': datetime.timedelta(days=7),

    'JWT_AUTH_HEADER_PREFIX': 'JWT',
    'JWT_AUTH_COOKIE': None,
}
```

用户可导入（调用）的数据
```python
IMPORT_STRINGS = (
    'JWT_ENCODE_HANDLER',
    'JWT_DECODE_HANDLER',
    'JWT_PAYLOAD_HANDLER',
    'JWT_PAYLOAD_GET_USER_ID_HANDLER',
    'JWT_PAYLOAD_GET_USERNAME_HANDLER',
    'JWT_RESPONSE_PAYLOAD_HANDLER',
    'JWT_GET_USER_SECRET_KEY',
)
```

#### APISettings
APISettings的路径：`rest-framework.settings`
`__getattr__`魔法方法的作用是，对象实例在调用方法或者属性时都将调用`__getattr__`。那么此处的作用，就相当于如果你想要调用`rest_framework_jwt.utils.jwt_payload_handler`函数，你就只需要使用`api_settings.JWT_PAYLOAD_HANDLER`即可。

```python
def __getattr__(self, attr):
	# defautls配置文件数据是最齐全的，如果defaults没有，就说明对象调用错了。
	# attr是属性名，或者方法名
	if attr not in self.defaults:
	    raise AttributeError("Invalid API setting: '%s'" % attr)

	try:
	    # 判断用户是否自定义了attr属性
	    val = self.user_settings[attr]
	except KeyError:
	    # 如果用户没有自定义，那么就从配置文件中读取
	    val = self.defaults[attr]

	# 如果attr在import_strings中，就说明其实是一个函数
	# 现在存入的是一个有具体路径的字符串，需要将其转换成可调用的函数
	if attr in self.import_strings:
	    val = perform_import(val, attr)

	# 加入缓存
	# 再次调用attr时，不会再触发__getattr__魔法方法
	self._cached_attrs.add(attr)
	# 这行代码，类似于这种效果
	# a = A() setattr(a, 'a', 10)
	# 打印 a.a 将返回10
	setattr(self, attr, val)
	return val
```

`rest_framework.settings`下也有个api_settings,
`api_settings = APISettings(None, DEFAULTS, IMPORT_STRINGS)`，与`rest_framework_jwt.settings`下的api_settings不是同一个，不要混用了。


#### jwt_payload_handler
路径：`rest_framework_jwt.utils.jwt_payload_handler`
```python
def jwt_payload_handler(user):
	...
	# 将用户id，用户名，token的有效时间，邮箱都写入载荷中
	payload = {
        'user_id': user.pk,
        'username': username,
        'exp': datetime.utcnow() + api_settings.JWT_EXPIRATION_DELTA
    }
    if hasattr(user, 'email'):
        payload['email'] = user.email
    if isinstance(user.pk, uuid.UUID):
        payload['user_id'] = str(user.pk)
	...
```

#### 加密与解密
```python
def jwt_encode_handler(payload):
    key = api_settings.JWT_PRIVATE_KEY or jwt_get_secret_key(payload)
    return jwt.encode(
        payload,
        key,
        api_settings.JWT_ALGORITHM
    ).decode('utf-8')


def jwt_decode_handler(token):
    options = {
        'verify_exp': api_settings.JWT_VERIFY_EXPIRATION,
    }
    # get user from token, BEFORE verification, to get user secret key
    unverified_payload = jwt.decode(token, None, False)
    secret_key = jwt_get_secret_key(unverified_payload)
    return jwt.decode(
        token,
        api_settings.JWT_PUBLIC_KEY or secret_key,
        api_settings.JWT_VERIFY,
        options=options,
        leeway=api_settings.JWT_LEEWAY,
        audience=api_settings.JWT_AUDIENCE,
        issuer=api_settings.JWT_ISSUER,
        algorithms=[api_settings.JWT_ALGORITHM]
    )
```

#### 用户自定义数据
以下是常用的几个，如果需要更多的，查看官方文档。
##### JWT_SECRET_KEY
用来给token加密的，默认是`settings.SECRET_KEY`

##### JWT_ALGORITHM
加密算法，默认是HS256

##### JWT_VERIFY
解密失败时将抛出一个DecodeError错误，默认是True，改为False时，仍可以获取载荷

##### JWT_VERIFY_EXPIRATION
是否设置有效时间，默认是True

##### JWT_LEEWAY
token过期时间的缓冲期，默认为0.就是说，如果设置了一个token的过期时间是10分钟，但如果你设置了这个值为1分钟，那么在10～11分钟内，该token仍有效。

##### JWT_EXPIRATION_DELTA
token的有效时间，默认是5分钟。`datetime.timedelta`数据类型。


### 参考
```text
http://getblimp.github.io/django-rest-framework-jwt/
https://www.django-rest-framework.org/api-guide/settings/
https://www.jianshu.com/p/180a870a308a
https://blog.csdn.net/chengqiang20152015/article/details/81146545
```