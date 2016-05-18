---
layout: 	post
title:		"小米手环R包与shiny应用"
header-img:	"img/post-bg-2016.jpg"
date:		2016-04-28
author: 	"Borg"
catalog:	true
tags:
    - 数据分析
---
花了好久写好的R包与shiny应用。R包可以在通过github安装，[repo](https://github.com/BigBorg/MiBand_R_Package)上也有安装说明，安装完成后运行runExample()能够调出本地版本的shiny app，app上的使用说明更加明确。不会R语言的可以直接无视package，有[shinyio](https://bigborg.shinyapps.io/MiBand/)上托管的应用，可以自己上传文件查看。注意上传文件是将目录 /data/data/com.xiaomi.hm.health/databases 压缩命名为databases.zip得到，注意是整个databases目录压缩，不是databases目录下的文件。用户id可以在手环官方应用个人信息页面得到。  

![shiny1](http://7xshuq.com1.z0.glb.clouddn.com//MiBand/img/2016-05-18%2022-09-42%E5%B1%8F%E5%B9%95%E6%88%AA%E5%9B%BE.png)  
![shiny2](http://7xshuq.com1.z0.glb.clouddn.com//MiBand/img/2016-05-18%2022-10-12%E5%B1%8F%E5%B9%95%E6%88%AA%E5%9B%BE.png)  
![shiny3](http://7xshuq.com1.z0.glb.clouddn.com//MiBand/img/2016-05-18%2022-10-29%E5%B1%8F%E5%B9%95%E6%88%AA%E5%9B%BE.png)  


