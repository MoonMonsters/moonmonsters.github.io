title: Django-rest-framework --- 权限
author: _Tao
tags:
  - django
  - django-rest-framework
categories:
  - Python
date: 2020-05-23 19:34:00
---
### 模型
```python
from django.db import models
from django.contrib.auth.models import User
 
 
class UserInfo(models.Model):
    USER_COMMON = 0
    USER_VIP = 100
    USER_SVIP = 200
    # 用户类型
    USER_TYPE = (
        (USER_COMMON, '普通用户'),
        (USER_VIP, 'VIP'),
        (USER_SVIP, 'SVIP')
    )
 
    user_type = models.IntegerField(choices=USER_TYPE, default=USER_COMMON)
    # 建立1对1关系模型
    user = models.OneToOneField(User, on_delete=models.CASCADE,
related_name='userinfo')
```

<!-- more -->

### 权限

```python
from rest_framework import permissions
 
from demo.models import UserInfo
 
 
class VipUserPermission(permissions.BasePermission):
    """
    VIP用户才可访问
    """
 
    def has_permission(self, request, view):
        return request.user.is_authenticated and
request.user.userinfo.user_type == UserInfo.USER_VIP
 
 
class SvipUserPermission(permissions.BasePermission):
    """
    SVIP用户才可访问
    """
 
    def has_permission(self, request, view):
        return request.user.is_authenticated and
request.user.userinfo.user_type == UserInfo.USER_VIP
```

### 视图
```python
from django.shortcuts import render
 
from django.contrib.auth.models import User
from django.contrib.auth import login
from rest_framework.views import APIView
from rest_framework.response import Response
from rest_framework import permissions
from demo.permissions import VipUserPermission, SvipUserPermission
 
 
class LoginApiView(APIView):
    # 任何人都可访问此视图
    permission_classes = []
 
    def get(self, request):
        return render(request, 'demo/index.html')
 
    def post(self, request):
        username = self.request.POST.get('username', '')
        password = self.request.POST.get('password', '')
 
        user = User.objects.filter(username=username).first()
        if user and user.password == password:
            login(request, user)
            return Response({'result': True})
 
        return Response({'result': False})
 
 
class CommonUserApiView(APIView):
    # 登录用户才可访问此视图
    permission_classes = [permissions.IsAuthenticated]
 
    def get(self, request):
        return Response({'result': True})
 
 
class VipUserApiView(APIView):
    # 交集关系,必须同时成立验证才通过
    # vip用户才可访问
    permission_classes = [VipUserPermission, SvipUserPermission]
 
    def get(self, request):
        return Response({'result': True})
```

### 全局配置
```python
REST_FRAMEWORK = {
    "DEFAULT_PERMISSION_CLASSES": ['demo.permissions.MyPermission'],  #
表示每一个视图类（只要不重写permission_classes属性），都需要拥有该权限用户才能访问。
}
```

### 总结
1. 通过权限,可控制用户可访问的视图
2.
可在配置文件中配置总权限,如果视图中没有重写permission_classes,那么就默认使用该权限
3. 如果不需要权限,那么permission_classes就设置为空
4. 如果视图的permission_classes中配置了多个权限,需要同时满足才能访问该视图

### 源代码

#### dispatch函数
会在dispatch函数中调用initial函数,初始化APIView中的某些属性

```python
def dispatch(self, request, *args, **kwargs):
    """
    `.dispatch()` is pretty much the same as Django's regular dispatch,
    but with extra hooks for startup, finalize, and exception handling.
    """
    ...
    request = self.initialize_request(request, *args, **kwargs)
    ...
 
    try:
        self.initial(request, *args, **kwargs)
 
        ...
        ....
```

#### initial函数
在initial函数中,会检查该视图需要的权限

```python
def initial(self, request, *args, **kwargs):
    """
    Runs anything that needs to occur prior to calling the method handler.
    """
    ...
    self.check_permissions(request)
    ...
```


#### check_permissions函数
在check_permissions函数中,会调用get_permissions()函数,得到Permissions对象列表,并调用各个对象下的has_permission()函数,
判断用户是否具有某权限,如果有一个不满足,则会调用permission_denied函数,给出错误消息提醒.

```python
def check_permissions(self, request):
    """
    Check if the request should be permitted.
    Raises an appropriate exception if the request is not permitted.
    """
    for permission in self.get_permissions():
        if not permission.has_permission(request, self):
            self.permission_denied(
                request, message=getattr(permission, 'message', None)
            )
```

#### permission_denied函数
message的意思是,如果在自定义的Permission类中定义了message的值,那么权限不通过时就会返回message的提示,
不再会采用默认的了.
```python
def permission_denied(self, request, message=None):
    """
    If request is not permitted, determine what kind of exception to raise.
    """
    if request.authenticators and not request.successful_authenticator:
        raise exceptions.NotAuthenticated()
    raise exceptions.PermissionDenied(detail=message)
 
class PermissionDenied(APIException):
    status_code = status.HTTP_403_FORBIDDEN
    default_detail = _('You do not have permission to perform this
action.')
    default_code = 'permission_denied'
```

### get_permissions函数
会遍历视图中重写的permission_classes列表并创建其中的类对象
```
def get_permissions(self):
    """
    Instantiates and returns the list of permissions that this view
requires.
    """
    return [permission() for permission in self.permission_classes]
```

如果视图没有重写该属性,将采用默认值,默认值就是在配置文件中REST_FRAMEWORK下配置的值
```python
permission_classes = api_settings.DEFAULT_PERMISSION_CLASSES
```

#### BasePermission类
若是继承了BasePermission类,但却没重写,两个函数都默认返回True
```python
@six.add_metaclass(BasePermissionMetaclass)
class BasePermission(object):
    """
    A base class from which all permission classes should inherit.
    """
 
    def has_permission(self, request, view):
        """
        Return `True` if permission is granted, `False` otherwise.
        """
        return True
 
    def has_object_permission(self, request, view, obj):
        """
        Return `True` if permission is granted, `False` otherwise.
        """
        return True
```

#### check_object_permissions
检查用户是否具有访问某单个对象的权限,跟check_permission一样.
如果重写了get_object函数,那么就需要手动调用,否则无法实现检查单个对象权限的功能
```python
def check_object_permissions(self, request, obj):
    """
    Check if the request should be permitted for a given object.
    Raises an appropriate exception if the request is not permitted.
    """
    for permission in self.get_permissions():
        if not permission.has_object_permission(request, self, obj):
            self.permission_denied(
                request, message=getattr(permission, 'message', None)
            )
```