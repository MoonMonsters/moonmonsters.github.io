title: Bug --- 多数据库使用外键错误
author: _Tao
tags:
  - django
categories:
  - Bug
date: 2020-05-23 19:21:00
---
### 前提
在项目中，A数据库中的某张表，想要关联上B数据库的User表，在使用`makemigrations`和`migrate`命令时都是正常的，数据表也可创建，但在插入数据，关联User对象时，报了错
```text
Unable to save with save_model using database router
```

<!-- more -->

### 出错原因

在stackoverflow上找到了同样的bug，解释如下：

The problem you are encountering arises from the difficulty of storing relations between objects that are stored in two distinct databases. In your example, you stated that you have created one database to store all Django contributed objects, which includes User objects created by the auth app. Meanwhile, the second model's objects will be stored in a distinct and entirely separate database. When you attempt to create a relationship between the new object stored in one database and the User object, you are attempting cross-database relations.

Cross-database relations are a difficult problem which has not been solved yet when using multiple databases in Django. If you would like more information about this issue, the Django documentation has a brief note about this problem (copied below for clarity).
>

    Django doesn’t currently provide any support for foreign key or many-to-many relationships spanning multiple databases. If you have used a router to partition models to different databases, any foreign key and many-to-many relationships defined by those models must be internal to a single database.
    
    This is because of referential integrity. In order to maintain a relationship between two objects, Django needs to know that the primary key of the related object is valid. If the primary key is stored on a separate database, it’s not possible to easily evaluate the validity of a primary key.
    
    If you’re using Postgres, Oracle, or MySQL with InnoDB, this is enforced at the database integrity level – database level key constraints prevent the creation of relations that can’t be validated.
    
    However, if you’re using SQLite or MySQL with MyISAM tables, there is no enforced referential integrity; as a result, you may be able to ‘fake’ cross database foreign keys. However, this configuration is not officially supported by Django.


​    
不想翻译，看的懂就行，总结一句话就是，django没有实现这个功能。


### 解决方案
...那么就不使用外键关联了，直接在A数据库的表中存入一个字段名为`user_id`的int类型数据即可，虽然之后的操作会稍微绕一下，但并不会影响功能的实现。


### 参考
[unable-to-save-with-save-model-using-database-router](https://stackoverflow.com/questions/26579231/unable-to-save-with-save-model-using-database-router)