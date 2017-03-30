---
layout: 	post
title:		"像素涂鸦应用总结之Ionic"
header-img:	"img/post-bg-deeplearning.jpg"
date:		2017-03-30
author: 	"Borg"
catalog:	true
tags:
    - Ionic
---
# Ionic

开头推销下刚写的应用啊，详见[github issue](https://github.com/driftyco/ionic/issues/5701)。大概就是底下这样的：  
![drawing](http://7xshuq.com1.z0.glb.clouddn.com/blog/img/2079268412.jpg)  
![svgcustomize](http://7xshuq.com1.z0.glb.clouddn.com/blog/img/947958209.jpg)  
![detail](http://7xshuq.com1.z0.glb.clouddn.com/blog/img/1443131507.jpg)  

正式开始介绍Ionic把。Ionic是一个跨平台的混合应用开发框架，基于Angularjs 。目前有第1版和第2版，分别基于 Angularjs 1和2。

如果是普通的应用，展示下信息，正常的应用交互，还是尽量使用新版Ionic 2。Ionic 2采用了typescript，因为扩展了javascript，提供了类型支持，编辑器的代码补全功能就变得异常强大。并且Ionic 2的官方[文档](http://ionicframework.com/docs/v2/native/)里包括了一堆像 3D touch, Alipay, Google map, Bluetooth, Geolocation等Native的功能，这些功能给应用留下了比较大的升级空间，虽然Ionic 1通过cordova插件的方式也能实现，但Ionic 2提供的从文档上看比较简单易用。并且angularjs 2的代码组织方式貌似也更模块化更好理解。  

不过呢，因为我的需求比较奇葩，并且一开始在coursera学的是Ionic 1, 所以一开始就是用Ionic 1写的。写到一半想换新版尝试改了下代码，发现我的需求还是比较适合用1来实现。需求大概就是一堆的SVG方块元素，需要支持缩放，触摸绘图。绘图的话如果是PC端有鼠标用mouseover事件很容易实现，但移动端并没有完全等效的事件。而我用的是Ionic 1中的drag事件，在一堆svg方块上划过的时候其实是在拖拽第一个元素，只不过第一个元素你没去改变它坐标它并不会真的被拖拽，但拖拽事件倒是随着手指移动不停触发，这样就能拿到手指的坐标了，而svg画布还要可缩放的话还需要用到Ionic 1里面的$ionPosition服务来定位画布位置，从而算出哪个方块被触摸到。就这个功能来说要移植到Ionic 2就比较难实现。首先缩放功能在Ionic 1中使用的是 &lt;ion-scroll&gt; 元素的zooming属性来实现的，虽然Ionic 2文档里也还有这个功能，还被命名为了zoom属性（去掉了ing），但其实它并不能如预期那样缩放，在 [github issue](https://github.com/driftyco/ionic/issues/6482) 里就能发现这个暂时还没人解决。还有Ionic 2也没有了$ionPosition服务和drag事件，虽然坐标还可以用其它方式获取，也可以使用touchover来替代drag事件，但我发现touchover被触发的频率相比于Ionic 1中的drag事件低很多，我们当然不希望用户划过去的时候有很多方块没有被填充，所以最后还是用了Ionic 1。  

还有就是如果需要使用第三方的js、css文件，Ionic 1 就比较容易实现，直接按平常那样在index页面引用就好了。而Ionic 2就比较麻烦，因为你编写的source文件夹并不是最后使用的，所以把文件放那并没有用。js其实还好，可以用npm安装，typescript里include进来就可以用了，而css则需要配置好让它能在typescript生成js的时候把文件复制过去。  

# Crosswalk

这个十分重要啊，用Ionic写应用如果不用crosswalk的话你会发现在电脑上用 Ionic serve --lab看是一个效果，但到了手机上就各种不行了，因为不同版本的系统用的浏览器不同，我测试的时候还发现虚拟机里的安卓还不能用Array.from()。而crosswalk则给你的应用里装了个浏览器，这样就不会有各种兼容问题了，就是安装包会比原来大一点，不过还是可以接受，我这个应用没用crosswalk之前貌似三兆左右安装包，用了后大概三十兆，压缩后也就二十几。

那么怎么用crosswalk呢，首先你肯定想到去查文档，我第一个试的也是文档上的，但你会发现，官方文档过时了！过时了！了！官方文档说的是用 ionic browser add crosswalk 命令，但实际上你会发现 ionic browser 命令已经不存在了！我最后在ionic的论坛里找到了答案，详见[该帖](https://forum.ionicframework.com/t/how-to-use-crosswalk-only-for-android/51065/2)。来自Ionic团队的回答是：  

use the plugin add command. The older browser add command was create before cordova had official support for crosswalk. Since then, cordova has added the parts needed to make this work, so we have depreciated the browser command.

现在安装crosswalk的方法是 cordova plugin add cordova-plugin-crosswalk-webview，或者cordova可以换成ionic也可以。  
最后吐槽下Ionic文档，文档简洁，预览功能很神奇很棒，但是一旦有问题文档真的是过于简洁了，而像这种过时的也是无奈了，可以去github和ionic的论坛里找找。  
