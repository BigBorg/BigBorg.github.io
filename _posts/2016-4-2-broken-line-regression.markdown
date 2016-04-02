---
layout: 	post
title:		"折线回归模型"
subtitle:	"Note: data science regression"
header-img:	"img/post-bg-deeplearning.jpg"
date:		2016-04-02
author: 	"Borg"
catalog:	true
tags:
    - 数据分析
    - 回归分析
    - 笔记
---
对于以下这样的数据应该如何用回归模型来预测呢？

```r
x <- -5:5
y <- c(5.12, 3.93, 2.67, 1.87, 0.52, 0.08, 0.93, 2.05, 2.54, 3.87, 4.97)
plot(x,y)
```

![plot of chunk unnamed-chunk-1](http://7xshuq.com2.z0.glb.clouddn.com/blog/img/figure/broken-line-regression-1.png) 

模型代码如下：

```r
knots<-c(0)
splineTerms<-sapply(knots,function(knot) (x>knot)*(x-knot))
xmat<-cbind(1,x,splineTerms)
xmat
```

```
##          x  
##  [1,] 1 -5 0
##  [2,] 1 -4 0
##  [3,] 1 -3 0
##  [4,] 1 -2 0
##  [5,] 1 -1 0
##  [6,] 1  0 0
##  [7,] 1  1 1
##  [8,] 1  2 2
##  [9,] 1  3 3
## [10,] 1  4 4
## [11,] 1  5 5
```

```r
fit<-lm(y~xmat-1)
yhat<-predict(fit)
plot(x,y)
lines(x,yhat,col="red")
```
xmat-1是因为已经把costant feature放到xmat里面了，就是xmat第一列。关键在于增加xmat[,3]该列，假设没有该列，只有原本的x和1,那么回归模型就是一条直线设为 y\=b+ax。加入splineTerms该列之后，由于当x\<0时对原模型无影响，仍是y\=b+ax，导数为a。而当x\>0时就变成了y\=b+ax+cx(x>=0时splineTerm等于x),对应导数也变为a+c，所以斜率改变。

![plot of chunk unnamed-chunk-2](http://7xshuq.com2.z0.glb.clouddn.com/blog/img/figure/broken-line-regression-2.png) 


