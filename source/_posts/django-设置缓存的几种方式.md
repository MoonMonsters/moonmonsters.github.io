title: Django --- 设置缓存的几种方式
author: _Tao
tags:
  - django
categories:
  - Python
date: 2020-05-23 19:22:00
---
### 缓存类型
#### memcached
`Memcached` 是一个高性能的分布式内存对象缓存系统。python使用它需要安装 `python-memcached` 依赖。 并且支持分布式缓存服务器:
```python
CACHES = {
    'default': {
        'BACKEND': 'django.core.cache.backends.memcached.MemcachedCache',
        'LOCATION': [
            '172.19.26.xxx:11211',
            '172.19.26.xxx:11211',
            '172.19.26.xxx:11213',
        ]
        # 'LOCATION': '127.0.0.1:11211', # 如果只有单台

    }
}
```
memchache的缓存完全是在内存中的，也就是，服务器一旦崩溃或重启，所有数据都不复存在，因此，决不能将缓存作为唯一的数据存储方式。

<!-- more -->

#### Redis
redis也是内存型缓存，但相比memcached，多了持久化的功能，更推荐使用这个。
```python
CACHES = {
    "default": {
        "BACKEND": "django_redis.cache.RedisCache",
        # "LOCATION": "redis://:xxx@192.168.0.7:6379/3",
        "LOCATION": "redis://127.0.0.1:6379/3",
        "OPTIONS": {
            "CLIENT_CLASS": "django_redis.client.DefaultClient",
        }
    }
}
```

#### 数据库
这种缓存方式就不怕断电丢失数据了，建表：
```python
python manage.py createcachetable [cache_table_name]
```
表名不要和其他冲突就行了，没什么需要注意的了。
```python
CACHES = {
    'default': {
        'BACKEND': 'django.core.cache.backends.db.DatabaseCache',
        'LOCATION': 'my_cache_table',
    }
}
```

#### 文件
文件路径需用 绝对路径，并且记得赋予读写权限。
```python
CACHES = {
    'default': {
        'BACKEND': 'django.core.cache.backends.filebased.FileBasedCache',
        'LOCATION': '/var/tmp/django_cache',
        #'LOCATION': 'c:/foo/bar',#windows下的示例
    }
}
```

#### 本地内存
如果有内存有点，但没能力运行memcache，就可以考虑采用本地内存缓存，这个缓存是多进程和线程安全的。
```python
CACHES = {
    'default': {
        'BACKEND': 'django.core.cache.backends.locmem.LocMemCache',
        'LOCATION': 'unique-snowflake'
    }
}
```
缓存LOCATION用来区分每个内存存储，如果你只有一个本地内存缓存，你可以忽略这个设置；但如果你有多个的时候，你需要至少给他们中一个赋予名字以区分他们。

注意每个进程都有它们自己的私有缓存实例，所以跨进程缓存是不可能的，因此，本地内存缓存不是特别有效率的，建议你只是在内部开发测时使用，不建议在生产环境中使用。

#### 虚拟缓存
不是真实的缓存，只是实现了缓存的接口而已。当如果你需要在开发和测试中不想使用缓存，但不想修改缓存相关的代码，这种情况就可以使用虚拟缓存了：
```python
CACHES = {
    'default': {
        'BACKEND': 'django.core.cache.backends.dummy.DummyCache',
    }
}
```

### 其他设置
有些缓存行为需要额外的参数来控制：

    TIMEOUT:缓存的默认过期时间，以秒为单位， 这个参数默认是 300 seconds (5 分钟).
    OPTIONS: 这个参数应该被传到缓存后端。有效的可选项列表根据缓存的后端不同而不同，由第三方库所支持的缓存将会把这些选项直接配置到底层的缓存库。
        MAX_ENTRIES:高速缓存允许的最大条目数，超出这个数则旧值将被删除. 这个参数默认是300.
        CULL_FREQUENCY:当达到MAX_ENTRIES 的时候,被删除的条目比率。 实际比率是 1 / CULL_FREQUENCY, 所以设置CULL_FREQUENCY 为2会在达到MAX_ENTRIES 所设置值时删去一半的缓存。这个参数应该是整数，默认为 3。 把 CULL_FREQUENCY的值设置为 0 意味着当达到MAX_ENTRIES时,缓存将被清空。
    KEY_PREFIX：将自动包含（默认情况下预置为）Django服务器使用的所有缓存键的字符串。
    VERSION：由Django服务器生成的缓存键的默认版本号。
    KEY_FUNCTION包含函数的虚线路径的字符串，定义如何将前缀，版本和键组成最终缓存键。



```python
CACHES = {
    'default': {
        'BACKEND': 'django.core.cache.backends.filebased.FileBasedCache',
        'LOCATION': '/var/tmp/django_cache',
        'TIMEOUT': 60,
        'OPTIONS': {
            'MAX_ENTRIES': 1000
        }
    }
}
```


### 参考
[缓存架构](http://hongweipeng.com/index.php/archives/1163/)