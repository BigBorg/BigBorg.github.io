---
layout: 	post
title:		"Django 博客系统"
header-img:	"img/post-bg-deeplearning.jpg"
date:		2017-04-25
author: 	"Borg"
catalog:	true
tags:
    - Django
---

跟着 [codingforentrepreneurs](http://codingforentrepreneurs.com) 网站上的三个系列教程从头到尾敲完了这个博客系统，并且对前端页面进行了些修改，还补了些Bug，总算是可以用了， github代码仓库在 [BigBorg/Blog](https://github.com/BigBorg/Blog)。已经部署到服务器，地址为 [bigborg.top](http://www.bigborg.top)，可以点开查看效果。总算不用为了发篇博文或者只是修改个拼写错误之类就来个git commit了。。。  

*codingforentrepreneurs* 教程最后版本的代码在·[github repo](https://github.com/codingforentrepreneurs/Blog-API-with-Django-Rest-Framework)， 不过不建议使用，因为里面真的有Bug。。。比如创建博文的时候 slug 的生成不支持中文！如果博文的标题中全是中文没有英文的话就会出现错误，创建的博文没有slug，进一步导致访问主页的时候会出错。我给补上了，用的 unidecode 包把中文转换成拼音生成slug。 

目前前端页面勉强还算可以， 以后再慢慢补，再增加博文标签等功能。以下介绍下该博客系统：

### 主页
![home][1]

主页是这样的 bootstrap 大屏风格，导航栏上有登录注册选项。而当用户登录后会变成注销，如果用户是管理员则还会有创建博文的选项。还有搜索栏，支持搜索博文哦！

![footer][2]

主页博文列表是有分页的，还有最低下的footer放置个人信息。

## 博文详情页

![detail][3]

博文详情页依然是这样的大屏风格，上面的背景图片可以在创建博文的时候上传，不上传则是如图的默认图片。左侧编辑和删除选项是只有以管理员登录才可见的，一般用户看不到的。

## 评论

![comment][4]

评论支持二级评论，评论的实现上使用了 Django Generic Foreign Key, 一个很神奇的东西。因为sql的外键是固定指向某个固定的字段的，而 GFK 则使得关系对象映射后的评论对象可以指向任何其它类型的ORM对象。如果不用GFK， 评论 Comment 类的对象就只能指向 Post 类的对象。但是使用了 GFK 后，Comment 对象就能够指向任意类型的对象。 也就是说，可以对博文作出评论，也可以对评论作出评论。总之 GFK 让 sql 貌似有了 nosql 的自由，不过数据库层面上的实现当然还是sql啦，只不过是多建了一张表用来存储所有的类类型，这样实现的。这个可以深究，后面可以再开个博文。

![GFK][5]

通过 Django 自带的 admin 管理系统可以看到评论确实是可以指向任意类型的对象的。

## 创建博文

![create][6]

创建博文是我最喜欢的部分，左侧是预览，右侧是填博文的表单。表单使用了crispyform， 所以才不会是丑的不行的 django 默认表单。左侧的预览使用了 js ， 把右侧输入的 markdown 实时地生成html预览。

## API

![API][7]

使用了 rest_framework 写 API 接口， 日后要写成应用也比较方便。 不得不说 rest_framework 真的是神器，十分方便。。。

# Bug Report
codingforentrepreneur 的代码是有 bug 的， 以下是我发现的：

- slug 生成不支持中文
- markdown 使用python转换html会出问题，尤其是代码块！！！
- 页面权限逻辑错误，在官方 repo 的 PR 里已经有人推送,不过看日期是16年4月推送的了，还没接受目测没人在维护了。

我的版本中以上 bug 都补上了，slug使用了中文拼音，markdown 使用 js 转换，权限只是几处or改成and。并且我使用了mysql数据库，python3 需要安装mysqlclient， requirement文件在代码仓库里。

  [1]: http://bigborg.top/static/img/blog/home.png
  [2]: http://bigborg.top/static/img/blog/footer.png
  [3]: http://bigborg.top/static/img/blog/detail.png
  [4]: http://bigborg.top/static/img/blog/comment.png
  [5]: http://bigborg.top/static/img/blog/genericFK.png
  [6]: http://bigborg.top/static/img/blog/create_post.png
  [7]: http://bigborg.top/static/img/blog/API.png
