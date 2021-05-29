title: Redis --- 分布式锁实现
author: _Tao
tags: []
categories:
  - Redis
date: 2021-02-24 23:16:00
---
### 什么是分布式锁
我们在开发应用的时候，如果需要对某一个共享变量进行多线程同步访问的时候，可以使用我们学到的锁进行处理，并且可以完美的运行，毫无Bug！

注意这是单机应用，后来业务发展，需要做集群，一个应用需要部署到几台机器上然后做负载均衡，大致如下图：
![](https://qxinhai.oss-cn-shenzhen.aliyuncs.com/hexo/20210224231757.png)

上图可以看到，变量A存在三个服务器内存中（这个变量A主要体现是在一个类中的一个成员变量，是一个有状态的对象），如果不加任何控制的话，变量A同时都会在分配一块内存，三个请求发过来同时对这个变量操作，显然结果是不对的！即使不是同时发过来，三个请求分别操作三个不同内存区域的数据，变量A之间不存在共享，也不具有可见性，处理的结果也是不对的！

如果我们业务中确实存在这个场景的话，我们就需要一种方法解决这个问题！

为了保证一个方法或属性在高并发情况下的同一时间只能被同一个线程执行，在传统单体应用单机部署的情况下，可以使用并发处理相关的功能进行互斥控制。但是，随着业务发展的需要，原单体单机部署的系统被演化成分布式集群系统后，由于分布式系统多线程、多进程并且分布在不同机器上，这将使原单机部署情况下的并发控制锁策略失效，单纯的应用并不能提供分布式锁的能力。为了解决这个问题就需要一种跨机器的互斥机制来控制共享资源的访问，这就是分布式锁要解决的问题！

分布式锁应该具备哪些条件：
> 

- 在分布式系统环境下，一个方法在同一时间只能被一个机器的一个线程执行；
- 高可用的获取锁与释放锁；
- 高性能的获取锁与释放锁；
- 具备可重入特性；
- 具备锁失效机制，防止死锁；
- 具备非阻塞锁特性，即没有获取到锁将直接返回获取锁失败

<!-- more -->

### 基于redis实现分布式锁

#### 选用Redis实现分布式锁原因

- Redis有很高的性能；
- Redis命令对此支持较好，实现起来比较方便

#### 使用命令介绍：

- SETNX

`SETNX key val`：当且仅当key不存在时，set一个key为val的字符串，返回1；若key存在，则什么都不做，返回0。

- expire

`expire key timeout`：为key设置一个超时时间，单位为second，超过这个时间锁会自动释放，避免死锁。

- delete

`delete key`：删除key

在使用Redis实现分布式锁的时候，主要就会使用到这三个命令。

#### 实现思想：

- 获取锁的时候，使用setnx加锁，并使用expire命令为锁添加一个超时时间，超过该时间则自动释放锁，锁的value值为一个随机生成的UUID，通过此在释放锁的时候进行判断。

- 获取锁的时候还设置一个获取的超时时间，若超过这个时间则放弃获取锁。

- 释放锁的时候，通过UUID判断是不是该锁，若是该锁，则执行delete进行锁释放。


### 代码实现

```python
import redis
import time
import uuid
from threading import Thread


class RedisLock(object):

    def __init__(self, client, lock_name, acquire_time=5, time_out=5):
        self._client: redis.Redis = client
        self._lock_name = lock_name
        self._acquire_time = acquire_time
        self._time_out = time_out

    def acquire(self):
        identifier = str(uuid.uuid4())
        # 最大获取锁的时间
        end = time.time() + self._acquire_time
        # 超过获取锁的最大时间, 那么获取锁失败
        while time.time() < end:
            # 判断是否获取锁成功
            if self._client.setnx(self._lock_name, identifier):
                # 设置锁的过期时间, 防止进程崩溃导致其他进程无法获取锁
                self._client.expire(self._lock_name, self._time_out)
                return identifier

            elif not self._client.ttl(self._lock_name):
                self._client.expire(self._lock_name, self._time_out)
            time.sleep(0.001)

        return False

    def release(self, identifier):
        pipe = self._client.pipeline(True)
        while True:
            try:
                # 用于监视一个(或多个) key ，
                # 如果在事务执行之前这个(或这些) key 被其他命令所改动，那么事务将被打断
                pipe.watch(self._lock_name)
                lock_value = self._client.get(self._lock_name)
                if not lock_value:
                    return True

                # 原子处理, 删除加锁值
                if lock_value.decode() == identifier:
                    # 用于标记一个事务块的开始
                    pipe.multi()
                    # 删除用来加锁的值
                    pipe.delete(self._lock_name)
                    # 事务块内的多条命令会按照先后顺序被放进一个队列当中，最后由 EXEC 命令原子性(atomic)地执行。
                    pipe.execute()
                    return True
                pipe.unwatch()
                break
            except redis.exceptions.WatchError:
                pass

        return False


count = 10


def test(thread_name):
    global count

    client = redis.Redis(host="localhost", port=6379, db=10)
    lock = RedisLock(client, "test-redis-lock")

    identifier = lock.acquire()
    print("线程:{}获取了锁".format(thread_name))
    time.sleep(1)
    if count < 1:
        print("线程:{}的count值小于1了".format(thread_name))
        return

    count -= 1
    print("线程:{}抢到了票, 还剩下{}张票".format(thread_name, count))
    lock.release(identifier)


if __name__ == '__main__':
    threads = [Thread(target=test, args=("thread-{}".format(i),)) for i in range(50)]
    for t in threads:
        t.start()

    for t in threads:
        t.join()

```

当然, 这个是自己手动实现的.
往上也已经有很好用的三方库可以直接使用了.
[python-redis-lock 3.7.0
](https://pypi.org/project/python-redis-lock/)



### 参考
[Python 使用 Redis 实现分布式锁](https://zhuanlan.zhihu.com/p/112016634)
[Redis分布式锁的python实现](https://www.cnblogs.com/chengege/p/11074055.html)
[python基于redis实现分布式锁](https://www.cnblogs.com/angelyan/p/11523846.html)