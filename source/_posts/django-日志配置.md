title: Django --- 日志配置
author: _Tao
tags:
  - django
categories:
  - Python
date: 2020-05-23 19:18:00
---
### 常用配置

```
import datetime

LOGGING = {
    # 指明dictConnfig的版本
    'version': 1,
    # 是否禁用所有的已经存在的日志配置
    'disable_existing_loggers': True,
    'formatters': {
        'standard': {
            # 配置打印log格式
            'format': '%(asctime)s [%(name)s:%(lineno)d] [%(module)s:%(funcName)s] [%(levelname)s] %(message)s'}
    },
    # 过滤器
    'filters': {
    },
    # 用来定义具体处理日志的方式，可以定义多种
    'handlers': {
        'mail_admins': {
            'level': 'ERROR',
            'class': 'django.utils.log.AdminEmailHandler',
            'include_html': True,
        },
        # 默认类型
        'default': {
            'level': 'DEBUG',
            'class': 'logging.handlers.RotatingFileHandler',
            # 保存的文件
            'filename': '{}/Log/QWebFX_{}.log'.format(BASE_DIR, datetime.datetime.now().date()),  # 日志输出文件
            # 文件大小
            'maxBytes': 1024 * 1024 * 5,
            # 备份份数
            'backupCount': 5,
            # 使用哪种formatters日志格式
            'formatter': 'standard',
        },
        'django_backends': {
            'level': 'DEBUG',
            'class': 'logging.handlers.RotatingFileHandler',
            # 保存的文件
            'filename': '{}/Log/backends_{}.log'.format(BASE_DIR, datetime.datetime.now().date()),  # 日志输出文件
            # 文件大小
            'maxBytes': 1024 * 1024 * 5,
            # 备份份数
            'backupCount': 5,
            # 使用哪种formatters日志格式
            'formatter': 'standard',
        },
        'error': {
            'level': 'ERROR',
            'class': 'logging.handlers.RotatingFileHandler',
            'filename': '{}/Log/Error/QWebFX_Error_{}.log'.format(BASE_DIR, datetime.datetime.now().date()),
            'maxBytes': 1024 * 1024 * 5,
            'backupCount': 5,
            'formatter': 'standard',
        },
        # 打印log到控制台
        'console': {
            'level': 'DEBUG',
            'class': 'logging.StreamHandler',
            'formatter': 'standard'
        },
        'request_handler': {
            'level': 'DEBUG',
            'class': 'logging.handlers.RotatingFileHandler',
            'filename': '{}/Log/Request/QWebFX_Request_{}.log'.format(BASE_DIR, datetime.datetime.now().date()),
            'maxBytes': 1024 * 1024 * 5,
            'backupCount': 5,
            'formatter': 'standard',
        },
        'scripts_handler': {
            # 只有INFO及以上级别的log才会写入文件中
            'level': 'INFO',
            'class': 'logging.handlers.RotatingFileHandler',
            'filename': '{}/Log/Script/QWebFX_Script_{}.log'.format(BASE_DIR, datetime.datetime.now().date()),
            'maxBytes': 1024 * 1024 * 5,
            'backupCount': 5,
            'formatter': 'standard',
        }
    },
    'loggers': {
        # django 表示就是django本身默认的控制台输出，就是原本在控制台里面输出的内容
        'django': {
            # 使用default类型的处理器进行处理
            'handlers': ['default'],
            'level': 'DEBUG',
            'propagate': False
        },
        'django.request': {
            'handlers': ['request_handler'],
            'level': 'DEBUG',
            'propagate': False,
        },
        'scripts': {
            'handlers': ['scripts_handler'],
            # 只有INFO及以上级别的消息才会调用此handler
            'level': 'INFO',
            'propagate': False
        },
        'console': {
            'handlers': ['console'],
            'level': 'DEBUG',
            'propagate': True
        },
        # API/Views 模块的日志处理
        'views': {
            'handlers': ['default', 'error'],
            'level': 'DEBUG',
            'propagate': True
        },
        'util': {
            'handlers': ['error'],
            'level': 'ERROR',
            'propagate': True
        },
        # 保存所有的数据库操作
        'django.db.backends': {
            'handlers': ['django_backends'],  # 指定file handler处理器，表示只写入到文件
            'level': 'DEBUG',
            'propagate': True,
        },
    }
}
```

<!-- more -->

使用方式:

```python
import logging
log = logging.getLogger('views')
log.debug('hello')
```
`getLogger`中传入的值会获取`loggers`中的数据，然后根据`handler`的值执行相应的操作。


### level:级别
<br/>
一个记录器是日志系统的一个实体，每一个记录器是一个已经命名好的可以将消息为进程写入的“桶”。
每一个记录器都会有一个日志等级，每个等级描述了记录器即将处理的信息的严重性，python定义了以下六个等级：
级别值描述
**CRITICAL 50** 关键错误/消息,描述已经发生的严重问题 <br/>
**ERROR 40** 错误,描述已经发生的主要问题 <br/>
**WARNING 30** 警告消息,描述已经发生的小问题 <br/>
**INFO 20** 通知消息,普通的系统信息列表内容 <br/>
**DEBUG 10** 调试,出于调试目的的低层次系统信息 <br/>
**NOTSET 0** 无级别<br/>

### 处理器/记录器 关键字参数
<br/>
**filename** 将日志消息附加到指定文件名的文件 <br/>
**filemode** 指定用于打开文件模式, 文件打开方式，在指定了filename时使用这个参数，默认值为“a”还可指定为“w”。 <br/>
**format** 用于生成日志消息的格式字符串 <br/>
**datefmt** 用于输出日期和时间的格式字符串 <br/>
**level** 设置记录器的级别 <br/>
**propagate** 可以基于每个记录器控制该传播。 如果您不希望特定记录器传播到其父项，则可以关闭此行为。 <br/>
**stream** 提供打开的文件，用于把日志消息发送到文件。可以指定输出到sys.stderr,sys.stdout或者文件，默认为sys.stderr。 <br/>
若同时列出了filename和stream两个参数，则stream参数会被忽略。

### format: 日志消息格式
<br/>
**%(name)s** 记录器的名称 <br/>
**%(levelno)s** 数字形式的日志记录级别 <br/>
**%(levelname)s** 日志记录级别的文本名称 <br/>
**%(filename)s** 执行日志记录调用的源文件的文件名称 <br/>
**%(pathname)s** 执行日志记录调用的源文件的路径名称 <br/>
**%(funcName)s** 执行日志记录调用的函数名称 <br/>
**%(module)s** 执行日志记录调用的模块名称 <br/>
**%(lineno)s** 执行日志记录调用的行号 <br/>
**%(created)s** 执行日志记录的时间 <br/>
**%(asctime)s** 日期和时间 <br/>
**%(msecs)s** 毫秒部分 <br/>
**%(thread)d** 线程ID <br/>
**%(threadName)s** 线程名称 <br/>
**%(process)d** 进程ID <br/>
**%(message)s** 记录的消息<br/>


### 内置处理器
<br/>
logging模块提供了一些处理器，可以通过各种方式处理日志消息。使用addHandler()方法将这些处理器添加给Logger对象。另外还可以为每个处理器配置它自己的筛选和级别。 <br/>
**logging.StreamHandler** 可以向类似与sys.stdout或者sys.stderr的任何文件对象(file object)输出信息 <br/>
**logging.FileHandler** 将日志消息写入文件filename。 <br/>
**logging.handlers.DatagramHandler(host，port)** 发送日志消息给位于制定host和port上的UDP服务器。使用UDP协议，将日志信息发送到网络 <br/>
**logging.handlers.HTTPHandler(host, url)** 使用HTTP的GET或POST方法将日志消息上传到一台HTTP 服务器。 <br/>
**logging.handlers.RotatingFileHandler(filename)** 将日志消息写入文件filename。如果文件的大小超出maxBytes制定的值，那么它将被备份为filenamel。 <br/>
**logging.handlers.SocketHandler** 使用TCP协议，将日志信息发送到网络。 <br/>
**logging.handlers.SysLogHandler** 日志输出到syslog <br/>
**logging.handlers.NTEventLogHandler** 远程输出日志到Windows NT/2000/XP的事件日志 
**logging.handlers.SMTPHandler** 远程输出日志到邮件地址 <br/>
**logging.handlers.MemoryHandler** 日志输出到内存中的制定buffer <br/>
注意：由于内置处理器还有很多，如果想更深入了解。可以查看官方手册。


### django提供的内置记录器
<br/>
**django** 在Django层次结构中的所有消息记录器。没有使用此名称发布消息，而是使用下面的记录器之一。 <br/>
**django.request** 与请求处理相关的日志消息。5xx响应被提升为错误消息；4xx响应被提升为警告消息。 <br/>
**django.server** 与由RunServer命令调用的服务器所接收的请求的处理相关的日志消息。HTTP 5XX响应被记录为错误消息，4XX响应被记录为警告消息，其他一切都被记录为INFO。 <br/>
**django.template** 与模板呈现相关的日志消息 <br/>
**django.db.backends** 有关代码与数据库交互的消息。例如，请求执行的每个应用程序级SQL语句都在调试级别记录到此记录器。<br/>


### 参考
```
https://docs.djangoproject.com/en/dev/topics/logging/#topic-logging-parts-formatters
https://www.jb51.net/article/161439.htm
https://blog.csdn.net/weixin_34416649/article/details/87073006
https://blog.csdn.net/haeasringnar/article/details/82053714
```