title: 使用gevent实现并发请求网络链接
author: _Tao
tags: []
categories:
  - python
date: 2020-05-23 19:23:00
---
#### 正常访问

耗时:`>>>total_time = 61.8569815158844`

```python
import requests
import time


def _urls():
    return [
               'https://www.baidu.com',
               'https://www.zhihu.com',
               'http://moonmonsters.pythonanywhere.com',
               'http://sina.com',
               'https://www.missshi.cn/api/view/blog/5a98e4fe5b925d0aae000005'
           ] * 5


def _request_url(url):
    return requests.get(url).content


def main():
    start = time.time()
    for url in _urls():
        _request_url(url)

    print('>>>total_time = ' + str(time.time() - start))


if __name__ == '__main__':
    main()

```

<!-- more -->

### 并发访问

耗时:`>>>total_time = 29.00442862510681`

对于gevent而言，我们需要准备一个任务列表。

任务列表中每一个元素都是一个gevent.spawn()对象。

其中，gevent.spawn()函数可以接收一至多个参数。

第一个参数为任务需要执行的函数，后续的参数为函数对应的输入参数。

最终，当我们得到完成的任务列表后，可以调用gevent.joinall()来执行该列表中的任务。

需要注意的是，当我们引入猴子补丁后，会对已经以后的方法进行改写。

因此，不建议在全局范围内引入猴子补丁，最好是在哪部分为并发执行函数，则在哪部分引入猴子补丁。

在引入猴子补丁后，当运行到网络请求时，则会切换至其他任务继续执行，而不是在当前任务中继续等待。

```python
import requests
import time
import gevent


def _urls():
    return [
               'https://www.baidu.com',
               'https://www.zhihu.com',
               'http://moonmonsters.pythonanywhere.com',
               'http://sina.com',
               'https://www.missshi.cn/api/view/blog/5a98e4fe5b925d0aae000005'
           ] * 5


def _request_url(url):
    # 猴子补丁
    from gevent import monkey
    monkey.patch_socket()
    return requests.get(url).content


def main():
    start = time.time()
    task_list = []
    for url in _urls():
        task_list.append(gevent.spawn(_request_url, url))

    gevent.joinall(task_list)

    print('>>>total_time = ' + str(time.time() - start))


if __name__ == '__main__':
    main()

```