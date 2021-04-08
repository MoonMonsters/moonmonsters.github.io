title: django - LocMemCache源码分析
author: _Tao
tags:
  - django
categories:
  - python
date: 2021-01-24 09:04:00
---
```python
"Thread-safe in-memory cache backend."

import time
from contextlib import contextmanager

from django.core.cache.backends.base import DEFAULT_TIMEOUT, BaseCache
from django.utils.synch import RWLock

try:
    from django.utils.six.moves import cPickle as pickle
except ImportError:
    import pickle


# Global in-memory store of cache data. Keyed by name, to provide
# multiple named local memory caches.
_caches = {}
_expire_info = {}
_locks = {}


@contextmanager
def dummy():
    """A context manager that does nothing special."""
    yield


class LocMemCache(BaseCache):
    def __init__(self, name, params):
        BaseCache.__init__(self, params)
        # 使用字典, 保存数据到内存
        # 按照_cull_frequency看, 此处可以设置成OrderDict, 并且按LRU方式, 删除较少访问的数据
        self._cache = _caches.setdefault(name, {})
        # 存储key过期信息
        self._expire_info = _expire_info.setdefault(name, {})
        # 使用了读写锁
        self._lock = _locks.setdefault(name, RWLock())

    def add(self, key, value, timeout=DEFAULT_TIMEOUT, version=None):
        """
        添加缓存数据
        """
        # 生成key值
        key = self.make_key(key, version=version)
        # 对key值校验
        self.validate_key(key)
        # 序列化
        pickled = pickle.dumps(value, pickle.HIGHEST_PROTOCOL)
        # 获取写锁
        with self._lock.writer():
            # 判断key值是否过期
            if self._has_expired(key):
                # key没有过期或者不存在, 则缓存数据
                self._set(key, pickled, timeout)
                return True
            return False

    def get(self, key, default=None, version=None, acquire_lock=True):
        key = self.make_key(key, version=version)
        self.validate_key(key)
        pickled = None
        # 获取读锁
        with (self._lock.reader() if acquire_lock else dummy()):
            if not self._has_expired(key):
                pickled = self._cache[key]
        # 获取到了数据
        if pickled is not None:
            try:
                # 反序列化
                return pickle.loads(pickled)
            except pickle.PickleError:
                return default

        with (self._lock.writer() if acquire_lock else dummy()):
            # 如果没有拿到数据, 则从缓存和过期信息中, 删除所有key值
            try:
                del self._cache[key]
                del self._expire_info[key]
            except KeyError:
                pass
            return default

    def _set(self, key, value, timeout=DEFAULT_TIMEOUT):
        """
        实际上的缓存操作
        """
        # 判断缓存的key值数量, 是否超过配置数量, 默认是300
        if len(self._cache) >= self._max_entries:
            #
            self._cull()
        self._cache[key] = value
        # 缓存过期时间信息
        self._expire_info[key] = self.get_backend_timeout(timeout)

    def set(self, key, value, timeout=DEFAULT_TIMEOUT, version=None):
        """
        缓存操作
        """
        key = self.make_key(key, version=version)
        self.validate_key(key)
        pickled = pickle.dumps(value, pickle.HIGHEST_PROTOCOL)
        with self._lock.writer():
            self._set(key, pickled, timeout)

    def incr(self, key, delta=1, version=None):
        """
        数值增加
        """
        with self._lock.writer():
            value = self.get(key, version=version, acquire_lock=False)
            if value is None:
                raise ValueError("Key '%s' not found" % key)
            # 没有确保value是数字判断?
            new_value = value + delta
            key = self.make_key(key, version=version)
            pickled = pickle.dumps(new_value, pickle.HIGHEST_PROTOCOL)
            self._cache[key] = pickled
        return new_value

    def has_key(self, key, version=None):
        """
        判断key值是否存在
        """
        key = self.make_key(key, version=version)
        self.validate_key(key)
        with self._lock.reader():
            if not self._has_expired(key):
                return True

        # 不存在则清空key值相关数据
        with self._lock.writer():
            # ...还不如直接调用self._delete(key)函数
            try:
                del self._cache[key]
                del self._expire_info[key]
            except KeyError:
                pass
            return False

    def _has_expired(self, key):
        """
        key值是否过期
        """
        # 获取过期时间
        exp = self._expire_info.get(key, -1)
        # 如果过期时间不存在(key值不存在的意思), 获取超过当前时间
        if exp is None or exp > time.time():
            return False
        return True

    def _cull(self):
        """
        删除频率
        """
        # 如果设置为0, 则清空所有缓存
        # 默认是3
        if self._cull_frequency == 0:
            self.clear()
        else:
            # 按取模操作, 按照设置的频率删除数据
            doomed = [k for (i, k) in enumerate(self._cache) if i % self._cull_frequency == 0]
            for k in doomed:
                self._delete(k)

    def _delete(self, key):
        try:
            del self._cache[key]
        except KeyError:
            pass
        try:
            del self._expire_info[key]
        except KeyError:
            pass

    def delete(self, key, version=None):
        key = self.make_key(key, version=version)
        self.validate_key(key)
        with self._lock.writer():
            self._delete(key)

    def clear(self):
        """
        清空所有缓存信息
        """
        self._cache.clear()
        self._expire_info.clear()

```