---
layout: 	post
title:		"Celery+Django: 异步邮箱验证"
header-img:	"img/post-bg-deeplearning.jpg"
date:		2017-05-08
author: 	"Borg"
catalog:	true
tags:
    - Django
    - Celery
---

其实。。。这篇教程不包括邮箱验证的，不过我有实现个 celery + django 的邮箱验证博客，问末附 repo 。

Web 应用中的长时操作如果没有异步实现会阻塞代码运行，用户需要等待较长时间才能收到响应。而像 Celery 这样的异步工具就能很好解决这类问题。本文将带你了解 Django 框架下的 Celery 使用。

# 安装

### Celery

Celery 使用 pip 安装即可。

```
pip install celery
```

Celery 还需要 **broker**，貌似有翻译成**中间人**的，类似个消息队列。这里将使用 Redis 作为 broker。redis 安装如下：

```python
wget http://download.redis.io/releases/redis-3.2.8.tar.gz
tar xzf redis-3.2.8.tar.gz
cd redis-3.2.8
make
make test
```

此时 src 文件夹下应该就有了 redis-server 和 redis-cli 二进制文件，可以把它们复制到 /usr/local/bin 目录下方便以后使用。此处暂不复制，启动 redis 服务器，使用默认端口 6379 。

```
redis-server
```
保持 redis 服务开启，生产环境下建议参考 [redis 入门指南][1] 进行 redis 配置。

### Django

Celery 4 支持 Django 1.8 以上版本，已经不需要安装其它包。

```
pip install django=1.9
```

# Django 项目
启动一个 Django 项目，并创建一个应用名为 Project ，创建一个应用叫 celeryExample。

```
django-admin startproject Project
cd Project
python manage.py startapp celeryExample
```
在 Project/Project 文件夹下创建一个文件用于放置 celery 启动的代码，名为 celery.py，在 Project/celeryExample 文件夹下创建一个文件用于放置具体的 celery task，名为 tasks.py。则文件结构应该如下：
- Project
  - Project
    - __init__.py
    - celery.py
  - celeryExample
    - __init__.py
    - tasks.py

为简洁，以上省去了 Django 自动创建的文件。  
把 celeryExample  添加到 Django INSTALLED_APPS 里。添加后 Project/Projcet/settings.py 中的 INSTALLED_APPS 应该如下：

```
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'celeryExample'
]
```

# Celery App
Django 环境下启动 Celery [官方文档][2]有相应代码，直接复制到 Project/Project/celery.py 文件内，稍加修改。此处将 'os.environ.setdefault` 第二个参数修改为“Project.settings”，将 celery 应用名称改为了 Project，在构造函数里传入 broker 参数设置使用 redis，并且删除了 debug_task 方法，最后代码如下，可直接复制。

```
from __future__ import absolute_import, unicode_literals
import os
from celery import Celery

# set the default Django settings module for the 'celery' program.
os.environ.setdefault('DJANGO_SETTINGS_MODULE', 'Project.settings')

app = Celery('Project', broker="redis://localhost")

# Using a string here means the worker don't have to serialize
# the configuration object to child processes.
# - namespace='CELERY' means all celery-related configuration keys
#   should have a `CELERY_` prefix.
app.config_from_object('django.conf:settings', namespace='CELERY')

# Load task modules from all registered Django app configs.
app.autodiscover_tasks()
```
上面的代码实例化了个 celery app，borker 参数可以设置使用的中间人，也不一定要使用redis，可以使用其它数据库。`app.autodiscover_tasks()` 能够自动在所有的 Django app 里查找 task，能够实现 celery app 和 task 的分离，更符合大型项目的结构。为保证 celery 能够在 Django 启动时启动，把 celery.py 里的 app import 到 Project/Project/__init__.py ，如下：
```
from .celery import app as celery_app
```

# Celery Tasks
Celery task 为具体的异步操作函数。在 Project/celeryExample/tasks.py 文件内进行定义。
```
from celery import shared_task
import time

@shared_task
def long_time_operation():
    time.sleep(10)
    return True
```
此处使用了 shared_task 装饰器，用于设置不与具体 celery 应用绑定的 task。使用`time.sleep(10)` 来模拟长时操作。  

有了 task 就可以使用它了，我们需要分别创建两个页面，一个使用异步操作，另一个则不使用异步操作进行比较。在 Project/celeryExample/views.py 里添加视图函数：

```
from celeryExample.tasks import long_time_operation
from django.http import HttpResponse
# Create your views here.

def async_op(request):
    long_time_operation.delay()
    return HttpResponse("Async Op!")

def block_op(request):
    long_time_operation()
    return HttpResponse("Finished!")
```
注意直接调用 `long_time_operation` 不会交给 celery 异步处理，需要使用 `long_time_operation.delay()' 才可以。

为视图函数添加相应的 url。修改 Project/Project/urls.py 如下：

```
from django.conf.urls import url, include
from django.contrib import admin

urlpatterns = [
    url(r'^admin/', admin.site.urls),
    url(r"^celery/", include("celeryExample.urls", namespace="celery"))
]
```

新建 Project/celeryExample/urls.py 如下：

```
from django.conf.urls import url
import celeryExample.views as views

urlpatterns = [
    url(r"async", views.async_op),
    url(r"block", views.block_op)
]
```

# 启动应用
首先启动 celery，在第一层 Project 文件夹下（与 manage.py 同一目录下）启动终端输入以下代码启动 celery。

```
celery -A Project worker --loglevel=info
```

接着可以启动 Django server 了，新开启一个终端输入：

```
python manage.py runserver
```

# 测试
此时就一切就绪了，首先访问`http://127.0.0.1:8000/celery/block`，浏览器将会一只处于加载状态，直到10秒后长时操作完成才加载结束返回“Finished!”。再访问`http://127.0.0.1:8000/celery/async`，由于使用了异步处理，浏览器立即就得到了响应，并且在运行 celery 的终端上有相应的输出如下：

```
[2017-05-08 15:49:41,244: INFO/MainProcess] Received task: celeryExample.tasks.long_time_operation[5ac8d68c-8c89-4c45-b779-66d12d285c02]  
[2017-05-08 15:49:51,251: INFO/PoolWorker-2] Task celeryExample.tasks.long_time_operation[5ac8d68c-8c89-4c45-b779-66d12d285c02] succeeded in 10.003452520999417s: True
```
是不是差异巨大！BTW: 我在我的博客系统的注册页面上使用了 celery 进行验证邮箱的发送，欢迎围观[github项目][3]，实际部署的[博客地址][4]，也欢迎 Pull Request！


  [1]: http://www.epubit.com.cn/Book/Details/1852
  [2]: http://docs.celeryq.org/en/latest/django/first-steps-with-django.html#using-celery-with-django
  [3]: https://github.com/BigBorg/Blog
  [4]: http://bigborg.top
