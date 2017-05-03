---
layout: 	post
title:		"Django Channels 实时在线用户列表"
header-img:	"img/post-bg-deeplearning.jpg"
date:		2017-05-03
author: 	"Borg"
catalog:	true
tags:
    - Django
    - WebSockets
---

本篇翻译自： [Getting Started with Django Channels][1]  
译者： Borg  
本篇教程中，我们将使用 Django Channels 来创建实时应用，当用户登录或登出时将实时更新已登录用户列表。

通过使用 WebSockets (Django Channels 实现) 来管理客户端与服务器的链接，当一个用户登录时，登录事件将被广播给所有已登录的用户。每个用户的界面不需要刷新即可自动更新。

注意： 本教程需要先熟悉 Django 和 WebSockets 的概念。 

### 本应用将使用：

    Python (v3.6.0)
    Django (v1.10.5)
    Django Channels (v1.0.3)
    Redis (v3.2.8)

### 目标  
你将能够：

 - 通过 Django Channels 为 Django 项目提供 WebSockets 支持。 
 - 让 Django 与 Redis 进行连接
 - 实现基本的用户验证 
 - 使用 Django 信号，当用户登录或登出时进行相应操作

### 开始
首先创建一个虚拟环境

    $ mkdir django-example-channels
    $ cd django-example-channels
    $ python3.6 -m venv env
    $ source env/bin/activate
    (env)$

安装 Django, Django Channels, ASGI Redis, 创建一个新的 Django 项目和应用。

    (env)$ pip install django==1.10.5 channels==1.0.2 asgi_redis==1.0.0
    (env)$ django-admin.py startproject example_channels
    (env)$ cd example_channels
    (env)$ python manage.py startapp example
    (env)$ python manage.py migrate

注意： 本教程将创建许多不同的文件和文件夹，如果忘记文件结构，可以参考[Github 代码仓库][2]。

接下来下载安装 Redis。如果你使用的是 Mac， 建议使用Homebrew:

`$ brew install redis`

在新的终端上启动 Redis 服务器，确保 Redis 使用默认端口 6379。 

更新 settings.py 的 INSTALLED_APPS :
```python
    INSTALLED_APPS = [
        'django.contrib.admin',
        'django.contrib.auth',
        'django.contrib.contenttypes',
        'django.contrib.sessions',
        'django.contrib.messages',
        'django.contrib.staticfiles',
        'channels',
        'example',
    ]
```
接着修改 CHANNEL_LAYERS， 配置默认后端和路由如下:
```python
CHANNEL_LAYERS = {
    'default': {
        'BACKEND': 'asgi_redis.RedisChannelLayer',
        'CONFIG': {
            'hosts': [('localhost', 6379)],
        },
        'ROUTING': 'example_channels.routing.channel_routing',
    }
}
```

### WebSockets 101

通常情况下， Django 使用 HTTP 实现客户端与服务器的交流。
 -  客户端发送 HTTP 请求给服务器
 - Django 处理请求，提取 URL ，与视图进行匹配
 - 试图处理请求并返回 HTTP 相应给客户端  

与 HTTP 不同， WebSockets 协议允许双向通信。用户未发出请求时，服务器也可以向客户端推送数据。 HTTP 中只有发出请求的客户端才能接收到响应。而 WebSockets 中，服务器可以同时与多个客户端通信。在 WebSockets 中，我们通过 ws:// 前缀来发送消息而非 http:// 。

注意： 建议先了解下 [Django Channel][3] 的定义。

### 消费者（Consumer）与群组（Group）

让我们创建第一个消费者，消费者将处理客户端与服务器间的链接。新建文件：example_channels/example/consumers.py:
```python
from channels import Group

def ws_connect(message):
    Group('users').add(message.reply_channel)

def ws_disconnect(message):
    Group('users').discard(message.reply_channel)
```
消费者与 Django 视图类似。任何连接的用户将会被添加到 users 群组，并且能接收到服务器发送的消息。当客户端断开链接，其相应的 channel 也将被从 users 群组中移除，用户也将停止接收消息。

接下来，让我们设置好路由。路由与 Django URL 设置几乎一样，新建文件example_channels/routing.py， 并写入以下代码：
```python
from channels.routing import route
from example.consumers import ws_connect, ws_disconnect

channel_routing = [
    route('websocket.connect', ws_connect),
    route('websocket.disconnect', ws_disconnect),
]
```
我们定义了 channel_routing 变量而非 urlpatterns， 使用了 route() 而非 url()。 注意此处我们讲消费者与 WebSockets 连接起来。

### 模板（Templates）
让我们写些 HTML 代码来与服务器进行 WebSocket 连接。在 example 文件夹内新建 templates 文件夹，接着在 templates 文件夹内创建 example 文件夹。文件路径： example_channels/example/templates/example 。

新建 _base.html 文件， HTML 代码如下：

```html
{% raw %}
<!doctype html>
<html lang="en">
<head>
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width, user-scalable=no, initial-scale=1.0, maximum-scale=1.0, minimum-scale=1.0">
  <meta http-equiv="X-UA-Compatible" content="ie=edge">
  <link href="//maxcdn.bootstrapcdn.com/bootstrap/3.3.7/css/bootstrap.min.css" rel="stylesheet">
  <title>Example Channels</title>
</head>
<body>
  <div class="container">
    <br>
    {% block content %}{% endblock content %}
  </div>
  <script src="//code.jquery.com/jquery-3.1.1.min.js"></script>
  {% block script %}{% endblock script %}
</body>
</html>
{% endraw %}
```

新建 user_list.html:

```html
{% raw %}
{% extends 'example/_base.html' %}

{% block content %}{% endblock content %}

{% block script %}
  <script>
    var socket = new WebSocket('ws://' + window.location.host + '/users/');

    socket.onopen = function open() {
      console.log('WebSockets connection created.');
    };

    if (socket.readyState == WebSocket.OPEN) {
      socket.onopen();
    }
  </script>
{% endblock script %}
{% endraw %}
```

现在，当客户端成功与服务器建立 WebSocket 连接时，我们将会在 console 中看到确认信息。

### 视图（Views）

创建视图来渲染模板，在 example_channels/example/views.py 中写入如下代码：
```python
from django.shortcuts import render

def user_list(request):
    return render(request, 'example/user_list.html')
```

在 example_channels/example/urls.py 设置好 URL:

```python
from django.conf.urls import url
from example.views import user_list

urlpatterns = [
    url(r'^$', user_list, name='user_list'),
]
```

在 example_channels/example_channels/urls.py 中更新 URL 如下:
```python
from django.conf.urls import include, url
from django.contrib import admin

urlpatterns = [
    url(r'^admin/', admin.site.urls),
    url(r'^', include('example.urls', namespace='example')),
]
```

### 测试
```shell
(env)$ python manage.py runserver
```
注意： 你可以选择在不同的终端运行 python manage.py runserver --noworker 和 python manage.py runworker 来将接口和 worker servers作为不同的进程进行测试。

当你访问 http://localhost:8000/， 你应当看到终端输出如下：

```shell
[2017/02/19 23:24:57] HTTP GET / 200 [0.02, 127.0.0.1:52757]
[2017/02/19 23:24:58] WebSocket HANDSHAKING /users/ [127.0.0.1:52789]
[2017/02/19 23:25:03] WebSocket DISCONNECT /users/ [127.0.0.1:52789]
```

### 用户验证
现在我们证明了可以打开一个链接，下一步是处理用户验证信息。记住：我们想要让用户能够登录，然后能看到所有订阅到 user 群组的其它用户。首先，我们需要用户能够创建账户并登录。我们将从登录页面开始。

在 example_channels/example/templates/example 新建一个 html 文件： log_in.html :
```html
{% raw %}
{% extends 'example/_base.html' %}

{% block content %}
  <form action="{% url 'example:log_in' %}" method="post">
    {% csrf_token %}
    {% for field in form %}
      <div>
        {{ field.label_tag }}
        {{ field }}
      </div>
    {% endfor %}
    <button type="submit">Log in</button>
  </form>
  <p>Don't have an account? <a href="{% url 'example:sign_up' %}">Sign up!</a></p>
{% endblock content %}
{% endraw %}
```

接下来更新 example_channels/example/views.py：

```python
from django.contrib.auth import login, logout
from django.contrib.auth.forms import AuthenticationForm
from django.core.urlresolvers import reverse
from django.shortcuts import render, redirect

def user_list(request):
    return render(request, 'example/user_list.html')

def log_in(request):
    form = AuthenticationForm()
    if request.method == 'POST':
        form = AuthenticationForm(data=request.POST)
        if form.is_valid():
            login(request, form.get_user())
            return redirect(reverse('example:user_list'))
        else:
            print(form.errors)
    return render(request, 'example/log_in.html', {'form': form})

def log_out(request):
    logout(request)
    return redirect(reverse('example:log_in'))
```

Django 表单支持验证。我们可以使用 AuthenticationForm 来处理用户登录。这一表单将检查用户名和密码，如果用户信息正确则会返回一个 User 对象。登录后用户将被重定向到主页。用户还应当能注销，所以我们新建了 logout 视图提供注销功能，并讲用户重定向到主页。

然后更新 example_channels/example/urls.py:
```python
from django.conf.urls import url
from example.views import log_in, log_out, user_list

urlpatterns = [
    url(r'^log_in/$', log_in, name='log_in'),
    url(r'^log_out/$', log_out, name='log_out'),
    url(r'^$', user_list, name='user_list')
]
```

我们还需要能够创建新用户。在 example_channels/example/templates/example 文件夹内创建 sing_up.html 如下：

```html
{% raw %}
{% extends 'example/_base.html' %}

{% block content %}
  <form action="{% url 'example:sign_up' %}" method="post">
    {% csrf_token %}
    {% for field in form %}
      <div>
        {{ field.label_tag }}
        {{ field }}
      </div>
    {% endfor %}
    <button type="submit">Sign up</button>
    <p>Already have an account? <a href="{% url 'example:log_in' %}">Log in!</a></p>
  </form>
{% endblock content %}
{% endraw %}
```

注意登录页面能够链接到注册页面，注册页面能够链接到登录页面。

添加以下视图：

```python
def sign_up(request):
    form = UserCreationForm()
    if request.method == 'POST':
        form = UserCreationForm(data=request.POST)
        if form.is_valid():
            form.save()
            return redirect(reverse('example:log_in'))
        else:
            print(form.errors)
    return render(request, 'example/sign_up.html', {'form': form})
```

我们使用另一内建表单来实现创建用户。表单成功验证后将重定向到登录页面。

记得要导入以下表单:

```python
from django.contrib.auth.forms import AuthenticationForm, UserCreationForm
```

再更新 example_channels/example/urls.py :

```python
from django.conf.urls import url
from example.views import log_in, log_out, sign_up, user_list

urlpatterns = [
    url(r'^log_in/$', log_in, name='log_in'),
    url(r'^log_out/$', log_out, name='log_out'),
    url(r'^sign_up/$', sign_up, name='sign_up'),
    url(r'^$', user_list, name='user_list')
]
```

现在，我们需要创建一个用户。运行服务器并访问 http://localhost:8000/sign_up/ 。填写有效的用户名密码创建第一个用户。

注意: 此处使用 michael 作为用户名， johnson123 作为密码。

注册视图将重定向到登录视图，在此我们可以用刚创建的用户进行登录。

新建多个用户以供下一步使用。

### 登录提示（Login Alert）

我们已经有了基本的用户验证，但还需要展示一个用户列表，并且需要服务器在用户登录登出时来告知群组。我们还需要修改消费者函数，让她们在客户端登录登出前发出消息。消息数据包括用户名和登录状态。

更新 example_channels/example/consumers.py :
```python
import json
from channels import Group
from channels.auth import channel_session_user, channel_session_user_from_http

@channel_session_user_from_http
def ws_connect(message):
    Group('users').add(message.reply_channel)
    Group('users').send({
        'text': json.dumps({
            'username': message.user.username,
            'is_logged_in': True
        })
    })

@channel_session_user
def ws_disconnect(message):
    Group('users').send({
        'text': json.dumps({
            'username': message.user.username,
            'is_logged_in': False
        })
    })
    Group('users').discard(message.reply_channel)
```

注意我们使用了装饰器来获取 Django session 的用户。同时，所有的消息都必须是 JSON 可序列化的，能够把数据转换为 JSON 字符串。

接下来更新 example_channels/example/templates/example/user_list.html:

```html
{% raw %}
{% extends 'example/_base.html' %}

{% block content %}
  <a href="{% url 'example:log_out' %}">Log out</a>
  <br>
  <ul>
    {% for user in users %}
      <!-- NOTE: We escape HTML to prevent XSS attacks. -->
      <li data-username="{{ user.username|escape }}">
        {{ user.username|escape }}: {{ user.status|default:'Offline' }}
      </li>
    {% endfor %}
  </ul>
{% endblock content %}

{% block script %}
  <script>
    var socket = new WebSocket('ws://' + window.location.host + '/users/');

    socket.onopen = function open() {
      console.log('WebSockets connection created.');
    };

    socket.onmessage = function message(event) {
      var data = JSON.parse(event.data);
      // NOTE: We escape JavaScript to prevent XSS attacks.
      var username = encodeURI(data['username']);
      var user = $('li').filter(function () {
        return $(this).data('username') == username;
      });

      if (data['is_logged_in']) {
        user.html(username + ': Online');
      }
      else {
        user.html(username + ': Offline');
      }
    };

    if (socket.readyState == WebSocket.OPEN) {
      socket.onopen();
    }
  </script>
{% endblock script %}
{% endraw %}
```

在主页面的用户列表，我们将用户名存为 data 属性，这样方便在 DOM 中获取。我们还对 WebSocket 添加了事件监听，这样能够处理服务器来的消息。我们将收到的消息转换为 JSON 数据，找到用户名对应的 <li> 元素，更新该用户的状态。

Django 不会追踪一个用户是否登录，所以我们需要新建一个模型来实现。在 example_channels/example/models.py 新建一个 LoggedInUser 模型, 用一对一关系与 User 模型相连：

```python
from django.conf import settings
from django.db import models

class LoggedInUser(models.Model):
    user = models.OneToOneField(
        settings.AUTH_USER_MODEL, related_name='logged_in_user')
```

我们的应用将会在用户登陆时创建一个 LoggedInUser 对象，当用户注销时会删除相应的对象。

执行 migrate 操作。

```shell
(env)$ python manage.py makemigrations
(env)$ python manage.py migrate
```

接下来更新 user list 视图来获取用户列表，在 example_channels/example/views.py:

```python
from django.contrib.auth import get_user_model, login, logout
from django.contrib.auth.decorators import login_required
from django.contrib.auth.forms import AuthenticationForm, UserCreationForm
from django.core.urlresolvers import reverse
from django.shortcuts import render, redirect

User = get_user_model()

@login_required(login_url='/log_in/')
def user_list(request):
    """
    NOTE: This is fine for demonstration purposes, but this should be
    refactored before we deploy this app to production.
    Imagine how 100,000 users logging in and out of our app would affect
    the performance of this code!
    """
    users = User.objects.select_related('logged_in_user')
    for user in users:
        user.status = 'Online' if hasattr(user, 'logged_in_user') else 'Offline'
    return render(request, 'example/user_list.html', {'users': users})

def log_in(request):
    form = AuthenticationForm()
    if request.method == 'POST':
        form = AuthenticationForm(data=request.POST)
        if form.is_valid():
            login(request, form.get_user())
            return redirect(reverse('example:user_list'))
        else:
            print(form.errors)
    return render(request, 'example/log_in.html', {'form': form})

@login_required(login_url='/log_in/')
def log_out(request):
    logout(request)
    return redirect(reverse('example:log_in'))

def sign_up(request):
    form = UserCreationForm()
    if request.method == 'POST':
        form = UserCreationForm(data=request.POST)
        if form.is_valid():
            form.save()
            return redirect(reverse('example:log_in'))
        else:
            print(form.errors)
    return render(request, 'example/sign_up.html', {'form': form})
```

如果一个用户有关联的 LoggedInUser，那么我们将该用户的 status 属性设置为 ”Online“， 反之则设置为 ”Offline“。我们还为用户列表视图和注销视图添加了 @login_required 装饰器来限制只有注册过的用户才能访问。

还有记得导入：

```python
from django.contrib.auth import get_user_model, login, logout
from django.contrib.auth.decorators import login_required
```

现在，用户可以登录登出，这将会触发服务器发送消息给客户端。但当用户刚登录的时候无法知道哪些用户登录了。用户只有在其它用户的状态改变时才能获取到更新。这时就需要用到 LoggedInUser。但我们需要在用户登录的时候创建 LoggedInUser 对象，当用户登出的时候删除它。

Django 本身包括了信号支持，能够在某一操作发生时广播通知。应用则可以监听这些信号，对此作出相应操作。我们可以使用两个内建的信号 user_logged_in 和 user_logged_out 来处理 LoggedInUser。

在 example_channels/example 添加一个新的文件 signals.py:

```python
from django.contrib.auth import user_logged_in, user_logged_out
from django.dispatch import receiver
from example.models import LoggedInUser

@receiver(user_logged_in)
def on_user_login(sender, **kwargs):
    LoggedInUser.objects.get_or_create(user=kwargs.get('user'))

@receiver(user_logged_out)
def on_user_logout(sender, **kwargs):
    LoggedInUser.objects.filter(user=kwargs.get('user')).delete()
```

我们需要使信号在应用设置里生效，修改 example_channels/example/apps.py :

```python
from django.apps import AppConfig

class ExampleConfig(AppConfig):
    name = 'example'

    def ready(self):
        import example.signals
```

更新 example_channels/example/__init__.py :

```python
default_app_config = 'example.apps.ExampleConfig'
```

### 测试

现在我们完成了代码，可以开始用多个帐号测试了。

运行 Django 服务器，登录并访问主页。我们应该看到应用内所有的用户组成的用户列表，每个用户都有个 Offline 的状态。接着打开一个新的隐私浏览器窗口（未登录），以另一个新用户的身份登录，注意查看两个窗口。当我们第二次登陆时，第一个浏览器窗口中相应用户的状态立即更新为 Online 。我们还可以在不同的设备上以不同的用户登录登出来进行测试。

![User List][4]

观察终端，我们将能确认 WebSocket 连接在用户登录时被创建，用户登出时被断开。

```shell
[2017/02/20 00:15:23] HTTP POST /log_in/ 302 [0.07, 127.0.0.1:55393]
[2017/02/20 00:15:23] HTTP GET / 200 [0.04, 127.0.0.1:55393]
[2017/02/20 00:15:23] WebSocket HANDSHAKING /users/ [127.0.0.1:55414]
[2017/02/20 00:15:23] WebSocket CONNECT /users/ [127.0.0.1:55414]
[2017/02/20 00:15:25] HTTP GET /log_out/ 302 [0.01, 127.0.0.1:55393]
[2017/02/20 00:15:26] HTTP GET /log_in/ 200 [0.02, 127.0.0.1:55393]
[2017/02/20 00:15:26] WebSocket DISCONNECT /users/ [127.0.0.1:55414]
```

注意： 你可以使用 [ngrok][5] 将本地服务器开放给互联网，这样就可以在其它设备上进行访问。

### 总结

本教程涉及了 Django Channels, WebSockets, 用户验证， 信号以及前端。最重要的是： Channels 扩展了传统 Django 应用，使服务器能够通过 WebSockets 给群组用户推送消息。

这很强大！

其应用可以有： 聊天室，多玩家游戏，即时通信的实时合作应用。甚至一些长时任务也可以用 WebSockets 进行改进。比如，不再需要间隔一段时间就对服务器发起一次访问来检查长时操作是否完成，而可以让服务器在完成操作后将状态更新推送给客户端。

本教程涉及的只是 Django Channels 所能做的冰山一角，可以查阅 [Django Channels][6] 文档看它还能做什么。

最终代码在 [django-example-channels repo][2]。


  [1]: https://realpython.com/blog/python/getting-started-with-django-channels/
  [2]: https://github.com/realpython/django-example-channels
  [3]: https://channels.readthedocs.io/en/stable/concepts.html
  [4]: https://realpython.com/images/blog_images/django-channels/django-channels-in-action.png
  [5]: https://ngrok.com/
  [6]: https://channels.readthedocs.io/en/stable/
