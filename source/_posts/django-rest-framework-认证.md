title: django-rest-framework --- 认证
author: _Tao
tags:
  - django
  - django-rest-framework
categories:
  - python
date: 2020-05-23 19:30:00
---
### 数据表

```python
class UserInfo(models.Model):
	USER_TYPE = (
		(1, '普通用户'),
		(2, 'VIP'),
		(3, 'SVIP')
	)

	user_type = models.IntegerField(choices=USER_TYPE, default=USER_TYPE[0][0])
	username = models.CharField(max_length=128)
	password = models.CharField(max_length=256)


class UserToken(models.Model):
	user = models.OneToOneField(UserInfo, on_delete=models.CASCADE)
	token = models.CharField(max_length=256)
```

<!-- more -->

### 自定义认证对象

```python
class CustomerAuthentication(authentication.BaseAuthentication):

	def authenticate(self, request):
		# 获取url中的token值
		token = request._request.GET.get('token')
		# 判断对应用户是否存在
		user_token = UserToken.objects.filter(token=token).first()

		if not user_token:
			raise exceptions.AuthenticationFailed('用户认证失败')

		# 必须返回一个包含两个元素的 tuple
		return (user_token.user, user_token)

	def authenticate_header(self, request):
		pass
```



### 视图

```python
from django.http import JsonResponse
from rest_framework import generics
from rest_framework.views import APIView

from students.models import UserInfo, UserToken
from students.authenticates import CustomerAuthentication


def md5(user):
	import hashlib
	import time

	ctime = str(time.time())

	m = hashlib.md5()
	m.update(ctime.encode('utf-8'))
	m.update(user.username.encode('utf-8'))

	return m.hexdigest()


class AuthApiView(APIView):

	def post(self, request, *args, **kwargs):
		ret = {'code': 200, 'msg': 'success'}

		try:
			username = request.POST.get('username')
			password = request.POST.get('password')
			user = UserInfo.objects.filter(username=username).first()

			if not user:
				UserInfo.objects.create(username=username, password=password)
				ret['code'] = 201
				ret['msg'] = '创建成功'

			token = md5(user)
			UserToken.objects.update_or_create(user=user, defaults={'token': token})
			ret['token'] = token

		except:
			import traceback
			traceback.print_exc()
			ret['code'] = 1002
			ret['msg'] = '请求异常'

		return JsonResponse(ret)


class OrderApiView(generics.ListAPIView):
	authentication_classes = [CustomerAuthentication, ]

	ORDER_DATA = {
		1: {
			'name': 'apple',
			'number': 20
		},
		2: {
			'name': 'banana',
			'number': 30
		}
	}

	def list(self, request, *args, **kwargs):
		return JsonResponse(OrderApiView.ORDER_DATA)
```







### 总结

当用户登录时，生成token并保存到数据表中，每次请求时都带上 token;

在视图类中重写authentication_classes属性，将自定义认证对象以列表形式赋值给他，当调用该视图时会自动验证；

如果想让所有的drf视图都使用同一个认证类，那么可以在settings.py中配置,

`'DEFAULT_AUTHENTICATION_CLASSES':['students.authenticates.CustomerAuthentication']`;

若不想使用settings.py中的认证，在视图类中重写authentication_classes属性即可。