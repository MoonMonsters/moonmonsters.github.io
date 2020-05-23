title: django --- 多数据库及分库实现
author: _Tao
tags:
  - django
categories:
  - python
date: 2020-05-23 19:19:00
---
在django项目中, 一个工程中存在多个APP应用很常见. 有时候希望不同的APP连接不同的数据库，这个时候需要建立多个数据库连接。

### 修改项目的 settings 配置

在settings.py中配置多个数据库，给个数据库有自己的功能
```python
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.mysql',
        'NAME': 'customs',
        'USER': 'xx',
        'PASSWORD': 'xx',
        'HOST': 'xx.xx.0.10',
        'PORT': '3306',
    },
    'xxx': {
        'ENGINE': 'django.db.backends.mysql',
        'NAME': 'xxx',
        'USER': 'root',
        'PASSWORD': 'xxx',
        'HOST': 'xx.xx.0.10',
        'PORT': '3306',
        'OPTIONS': {
            'init_command': 'SET storage_engine=MyISAM',
        },
    }
}
```

<!-- more -->

### 设置数据库的路由规则方法

在 settings.py 中配置 DATABASE_ROUTERS
```python
# 数据库分库
# User, Group, HumanRole三张表使用xxx的数据库，其他的使用默认的
# DatabaseAppsRouter是路由类的路径
DATABASE_ROUTERS = ['proutils.db_router.DatabaseAppsRouter']
# 每个APP要连接哪个数据库，需要在做匹配设置
# 如果没有配置的，将使用default数据库
DATABASE_APPS_MAPPING = {
    'auth': 'xxx',
    'xxx_human': 'xxx',
}
```

### 创建数据库路由规则
```python
# !/usr/bin/env python
# -*- coding:utf-8 -*-
# __author__ = 'ChenTao'


from django.conf import settings

DATABASE_MAPPING = settings.DATABASE_APPS_MAPPING


class DatabaseAppsRouter(object):
    """
    数据库分库设置
    """
    """
    A router to control all database operations on models for different
    databases.

    In case an app is not set in settings.DATABASE_APPS_MAPPING, the router
    will fallback to the `default` database.

    Settings example:

    DATABASE_APPS_MAPPING = {'app1': 'db1', 'app2': 'db2'}
    """

    def db_for_read(self, model, **hints):
        """"Point all read operations to the specific database."""
        if model._meta.app_label in DATABASE_MAPPING:
            return DATABASE_MAPPING[model._meta.app_label]
        return None

    def db_for_write(self, model, **hints):
        """Point all write operations to the specific database."""
        if model._meta.app_label in DATABASE_MAPPING:
            return DATABASE_MAPPING[model._meta.app_label]
        return None

    def allow_relation(self, obj1, obj2, **hints):
        """Allow any relation between apps that use the same database."""
        db_obj1 = DATABASE_MAPPING.get(obj1._meta.app_label)
        db_obj2 = DATABASE_MAPPING.get(obj2._meta.app_label)
        if db_obj1 and db_obj2:
            if db_obj1 == db_obj2:
                return True
            else:
                return False
        return None

    def allow_syncdb(self, db, model):
        """Make sure that apps only appear in the related database."""

        if db in DATABASE_MAPPING.values():
            return DATABASE_MAPPING.get(model._meta.app_label) == db
        elif model._meta.app_label in DATABASE_MAPPING:
            return False
        return None

    def allow_migrate(self, db, app_label, model=None, **hints):
        """
        Make sure the auth app only appears in the 'auth_db'
        database.
        """
        if db in DATABASE_MAPPING.values():
            return DATABASE_MAPPING.get(app_label) == db
        elif app_label in DATABASE_MAPPING:
            return False
        return None

```

### Models创建样例
例如:
```python
class HumanRole(models.Model):
    ...
    ...

    class Meta:
        app_label = 'xxx_human'
```
之后在路由分发的时候，会根据app_label的值从`DATABASE_APPS_MAPPING`得到不同的数据库名，然后使用。


### 生成数据表
在使用django的 migrate 创建生成表的时候，需要加上 –database 参数，如果不加则将 未 指定 app_label 的 APP的models中的表创建到default指定的数据库中,如:

```python
python manage.py  migrate account --database=db01
```
以上创建完成后,其它所有的创建、查询、删除等操作就和普通一样操作就可以了,无需再使用类似
`models.User.objects.using(dbname).all(）`
这样的方式来操作