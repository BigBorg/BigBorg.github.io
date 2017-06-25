---
layout: 	post
title:		"robotparser 与 requests 结合使爬虫遵守 robots.txt 协议"
header-img:	"img/post-bg-deeplearning.jpg"
date:		2017-06-26
author: 	"Borg"
catalog:	true
tags:
    - Python
    - Crawler
---

# 什么是 robots.txt ?
robots.txt 文件放置在网站根目录下，定义了什么样的客户端（web服务器以User-Agent识别客户端）可以访问的资源有哪些，不能访问的资源有哪些。以百度的 robots.txt 为例，如下:

    User-agent: Baiduspider
    Disallow: /baidu
    Disallow: /s?
    Disallow: /ulink?
    Disallow: /link?
    
    User-agent: *
    Disallow: /

截取的两部分，第一部分定义了百度自家的爬虫 Baiduspider 不允许爬取的内容，而第二部分则说明没有 User-Agent 的客户端任何资源都不允许访问。

# robotparser
robotparser 是 python 自带的库，能够用于解析 robots.txt 规则，判断要爬取的 url 按照 robots.txt 文件是否合法。 robotparser 官方的文档： [robotparser doc][1]. 官方文档内容还是比较少的，经测试发现 RobotFileParser 的几个行为：

 1. set_url 方法并不是只能用于一个站点，而是可以连续添加多个站点，而之前添加的依然有效。
 2. 当传递给 set_url 方法的参数并不是一个真实的 robots.txt 文件地址时并不会报错。
3. 对于未添加过 robots.txt 文件的站点，can_fetch 方法默认返回 False。

# 目标
我们希望在使用 requests 的 Session 实例能够添加 robots.txt 文件，并在使用 get 和 post 方法时如果 robots.txt 规则不允许则抛出异常。

# RobotsSession
为了实现上述功能，以下代码继承了 requests.Session ，实现了 add_robot 方法和 allowed 方法。add_robot 方法接受一个 url 参数，提取出站点相应的 robots.txt 地址，再用传给 RobotFileParser 实例进行解析，同时记录添加过的站点。allowed 方法则用于判断当前解析到的 robots.txt 文件规则下爬取参数 url 是否合法。但对于没有添加过 robots.txt 文件的站点，我们希望默认可以爬取，因此在此比对已添加的站点，如果待爬取的站点没有添加 robots.txt 文件，则返回True，只有添加过的站点才按规则解析。

    class RobotsSession(Session):
    
        def __init__(self, *args, **kwargs):
            if kwargs.has_key('follow_robots'):
                self._follow_robots = kwargs.get("follow_robots")
                del kwargs['follow_robots']
            else:
                self._follow_robots = True
    
            self._robot = RobotFileParser() # host to robotparser obj
            self._robot_hosts = Set()       # hosts added
    
            super(RobotsSession, self).__init__(*args, **kwargs)
    
        def add_robot(self, url):
            '''
            any url to be crawled, this method will convert url for robots file
            :param url: 
            :return: 
            '''
            host = RobotsSession.url2schemahost(url)
            robot_path = host + "/robots.txt"
            if not host in self._robot_hosts:
                self._robot_hosts.add(host)
                self._robot.set_url(robot_path)
                self._robot.read()
            return True
    
        def allowed(self, url):
            '''
            :param url: 
            :return: 
            '''
            if not self._follow_robots:
                return True
            host = RobotsSession.url2schemahost(url)
            if not host in self._robot_hosts:
                return True
            return self._robot.can_fetch(self.headers['User-Agent'], url)
    
        @staticmethod
        def url2schemahost(url):
            components = urlparse(url)
            return components.scheme + "://" + components.netloc

# get， post 方法
使用装饰器，重写 get, post方法，检查 url 是否合法，不合法抛出异常，合法照常爬取。

    class RobotsNotAllowError(Exception):
        def __init__(self, *args, **kwargs):
            super(RobotsNotAllowError, self).__init__("This url is not allowed for crawling.", *args, **kwargs)
    
    def follow_robots(func):
        def wrapper(instance, url, *args, **kwargs):
            if instance.allowed(url):
                return func(instance, url, *args, **kwargs)
            raise RobotsNotAllowError()
        return wrapper
    
    class RobotsSession(Session):
        @follow_robots
        def post(self, *args, **kwargs):
            return super(RobotsSession, self).post(*args, **kwargs)
    
        @follow_robots
        def get(self, *args, **kwargs):
            return super(RobotsSession, self).get(*args, **kwargs)

# 全部代码、使用及将来可能的改进
全部的代码如下，使用前需要预先添加 robots.txt 文件。因为 robotparser 遇到错误的 robots.txt 路径并不会报错，比较难判断一个站点是否真的有 robots 文件，特别是很多站点没有 robots 返回的状态码还是 200 ，而对这些站点 robotparser 默认不允许爬取，而我们又希望默认可以爬取，所以就先写成预先手动添加站点 robots 路径的形式。如果能明确判断一个站点是否有 robots.txt 文件，则可以改进成无需添加自动解析的形式。

    #coding:utf-8
    
    from robotparser import RobotFileParser
    from urlparse import urlparse
    from requests import Session
    from sets import Set
    
    class RobotsNotAllowError(Exception):
        def __init__(self, *args, **kwargs):
            super(RobotsNotAllowError, self).__init__("This url is not allowed for crawling.", *args, **kwargs)
    
    def follow_robots(func):
        def wrapper(instance, url, *args, **kwargs):
            if instance.allowed(url):
                return func(instance, url, *args, **kwargs)
            raise RobotsNotAllowError()
        return wrapper
    
    class RobotsSession(Session):
    
        def __init__(self, *args, **kwargs):
            if kwargs.has_key('follow_robots'):
                self._follow_robots = kwargs.get("follow_robots")
                del kwargs['follow_robots']
            else:
                self._follow_robots = True
    
            self._robot = RobotFileParser() # host to robotparser obj
            self._robot_hosts = Set()       # hosts added
    
            super(RobotsSession, self).__init__(*args, **kwargs)
    
        def add_robot(self, url):
            '''
            any url to be crawled, this method will convert url for robots file
            :param url: 
            :return: 
            '''
            host = RobotsSession.url2schemahost(url)
            robot_path = host + "/robots.txt"
            if not host in self._robot_hosts:
                self._robot_hosts.add(host)
                self._robot.set_url(robot_path)
                self._robot.read()
            return True
    
        def allowed(self, url):
            '''
            :param url: 
            :return: 
            '''
            if not self._follow_robots:
                return True
            host = RobotsSession.url2schemahost(url)
            if not host in self._robot_hosts:
                return True
            return self._robot.can_fetch(self.headers['User-Agent'], url)
    
        @follow_robots
        def post(self, *args, **kwargs):
            return super(RobotsSession, self).post(*args, **kwargs)
    
        @follow_robots
        def get(self, *args, **kwargs):
            return super(RobotsSession, self).get(*args, **kwargs)
    
        @staticmethod
        def url2schemahost(url):
            components = urlparse(url)
            return components.scheme + "://" + components.netloc
    
    if __name__ == "__main__":
        session = RobotsSession(follow_robots=True)
        print session.get("http://www.baidu.com").status_code   # proceed
    
        session.add_robot("http://www.baidu.com")
        try:
            session.get("http://www.baidu.com")  # Should fail
        except RobotsNotAllowError as error:
            print error
    
        print session.get("http://www.weibo.com").status_code  # proceed
    
        session.add_robot("http://www.weibo.com")
    
        try:
            session.get("http://www.weibo.com")  # Should fail
        except RobotsNotAllowError as error:
            print error
    
        print "Test passed"

  [1]: https://docs.python.org/2/library/robotparser.html
