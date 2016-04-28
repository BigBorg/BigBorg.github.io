---
layout: 	post
title:		"安装Ubuntu后要做的几件事"
header-img:	"img/post-bg-2016.jpg"
date:		2016-04-28
author: 	"Borg"
catalog:	true
tags:
    - Ubuntu
---
# 安装Ubuntu之后做的几件事：  
对的，我换电脑了！换了个8G内存宏碁的笔记本，这样就能搞得起某些数据分析的算法了。
买来新电脑第一件事当然是装系统啦，但是装完系统后好多软件需要重新安装。虽然说Ubuntu可以直接导出软件源和已安装软件列表，但是发现自己好多软件就不是用apt-get安装的。所以在此记录下本次装系统所做修改，以备不时之需。  
首先是Ubuntu版本的选择，现在Ubuntu已经有了16.04版本，就选了最新的。但是，一定要选麒麟版本！第一次觉得之前碰到些错误信息是中文的不方便谷歌，所以没选麒麟，结果中文输入法装的真的很麻烦！本来想装搜狗的中文输入法的，结果各种依赖关系不满足，而且有些还找不到软件源。。。总之各种麻烦，相比之下麒麟已经预装了中文输入法，还有中文农历的日历（虽然用不上），还更换了原本的主题（有点花哨，换回了原来的），把unity从左边换到了底部（16.04新特性哦，不过有些软件界面下部可能会被挡住，为了怀旧换回了左侧哈哈）。16.04把以前的应用市场换成了软件中心，界面更清爽速度更快，但是刚打开没有更新软件源真的是什么都没有啊，而麒麟的Ubunt Kylin软件中心还不错，推荐的也都很适合中国用户，包括整理了win对应的ubuntu替换软件，还有wineQQ什么的。。。总之还是不要做，选麒麟最省事了，省得自己得改半天。  

# Restricted Extras、build-essentials
非开源的第三方视频音频解码器，还有编译所需必要的库，不过可能在安装的时候已经勾选了，再输入一遍也没坏处。。。

```bash
sudo apt-get install ubuntu-restricted-extras build-essentials
```

# 点击Unity launcher软件图标窗口最小化
新版功能

```
gsettings set org.compiz.unityshell:/org/compiz/profiles/unity/plugins/unityshell/ launcher-minimize-window true
```

# unity-tweak-tool
用于更改主题的，各种个性化，还是很推荐的。

```
sudo apt-get install unity-tweak-tool
```

# PPPoE
之前Ubuntu因为NetworkManager和interfaces的冲突，没法在wifi下用pppoe登录宽带。学校的电信宽带按时计费，校园网太慢，所以偶尔需要用到电信。
修改以下文件：

```
/etc/NetworkManager/NetworkManager.conf  
```

```
[ifupdown]  
managed=false 
```
false改成true，然后就可以用pppoeconf设置拨号帐号了。

# ss你懂的，哈哈
之前14.04用的pip安装的shadowsocks，但现在安装了之后并没有sslocal命令，而apt安装的不支持rd4-md5加密，于是找到了如下的github项目：
[https://github.com/shadowsocks/shadowsocks-libev](https://github.com/shadowsocks/shadowsocks-libev)
该项目的apt-get安装方式不可行，因为key下不下来，估计被封了。所以只能下载源码，编译安装，具体项目reademe已经写的很清楚了。最后用的时候注意是ss-local而不是sslocal了。

# R
r-core需要添加软件源，还需要key，命令如下

```bash
sudo gedit /etc/apt/sources.list
```
在最后加上

```
deb http://mirrors.opencas.cn/cran/bin/linux/ubuntu trusty/
```
这行，便是添加软件源咯，不过还需要添加key才能安装。对了，16.04的apt-get已经可以不用-get了，就是直接apt就等同于之前的apt-get，同时apt-get还可以用。

```
gpg --keyserver hkp://keyserver.ubuntu.com:80 --recv-keys E084DAB9  
gpg -a --export E084DAB9 | sudo apt-key add -  
sudo apt-get upgrade  
sudo apt-get install r-base  
```

# rstudio
Rstudio到官网下载deb包，dpkg安装，不详细说明啦。官网地址：[https://www.rstudio.com/](https://www.rstudio.com/)

# Anaconda, ipython notebook, graphlab create
在学习华盛顿大学coursera上的机器学习课程（强烈推荐），所以需要这些。
## Anaconda
[Anaconda](https://www.continuum.io/downloads)直接去官网下载相应的sh文件直接执行就好了，注意需要先把文件chmod +x增加执行权限，执行时不需要sudo。

## graphlab create
graphlab create可以在线申请免费使用一年，一年后可以用同样的邮箱再次申请，简直不能再棒。可以去官网[dato](https://dato.com/)申请，具体安装过程都按照相应的帐号密码生成了对应的命令，直接去复制就好了,注意最低下有升级，用GPU提升计算的。

## ipython notebook
```
sudo apt-get install ipython
```

# Jekyll
用于github page生成博客的。

```
sudo apt-get install ruby ruby-dev
sudo gem install jekyll jekyll-paginate
```


