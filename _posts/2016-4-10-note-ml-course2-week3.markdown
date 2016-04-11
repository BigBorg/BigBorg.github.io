---
layout: 	post
title:		"评估模型"
subtitle:	"Note: ml regression week3"
header-img:	"img/post-bg-deeplearning.jpg"
date:		2016-04-10
author: 	"Borg"
catalog:	true
tags:
    - 数据分析
    - 回归分析
    - 笔记
---
# 如何评估一个回归模型  

## loss function
首先定义loss function，即预测错误带来的损失，通常使用y-yhat的绝对值或者平方。但不绝对，比如在预测放假时如果估值过高，则可能完全卖不出去，带来的损失更大，因此可以定义loss function使估值过高带来的loss比估值过低带来的loss更大。
![loss function](http://7xshuq.com2.z0.glb.clouddn.com/blog/img/figure/coursera/ml-washington/wk3/1.png)   

然后看到一句很有意思的话：
![quote](http://7xshuq.com2.z0.glb.clouddn.com/blog/img/figure/coursera/ml-washington/wk3/2.png)  

## training error, generalization error, test error  

### training error
training error的计算方式：
![quote](http://7xshuq.com2.z0.glb.clouddn.com/blog/img/figure/coursera/ml-washington/wk3/3.png)  

以下是以suare error为例（再次强调是loss function，不一定是error的平方）：
![quote](http://7xshuq.com2.z0.glb.clouddn.com/blog/img/figure/coursera/ml-washington/wk3/4.png)  

随着模型复杂度增加，traing error会不断减少
![quote](http://7xshuq.com2.z0.glb.clouddn.com/blog/img/figure/coursera/ml-washington/wk3/5.png)  

### generalization error
generalization error的定义，注意对于每一对(x,y)要乘上相应的概率。就是说，generlization error是对于每一个可能的(x,y)求loss，即整体population的loss的期望值。举例来说，现实生活中的房屋面积分布，离均值越远，概率越小，所以对loss的期望值影响也应当越小。
![quote](http://7xshuq.com2.z0.glb.clouddn.com/blog/img/figure/coursera/ml-washington/wk3/6.png)  

那么实际上由于现实生活中的房屋不能全部调查，所以generalization error是没法计算的。不过理论上，随着模型复杂度增加，首先呈减小趋势，随后由于overfitting，generlization error又增加。
![quote](http://7xshuq.com2.z0.glb.clouddn.com/blog/img/figure/coursera/ml-washington/wk3/8.png)  

### test error
test error就不用说了，跟training error一样，只是用test data计算。由于generalization error无法计算，所以可以用test error估计。由于是估计，所以会有偏差，于是就是沿着generalization error上下波动了。
![quote](http://7xshuq.com2.z0.glb.clouddn.com/blog/img/figure/coursera/ml-washington/wk3/9.png)  

## 3 source of error
error有三个来源：noise, bias, variance
![quote](http://7xshuq.com2.z0.glb.clouddn.com/blog/img/figure/coursera/ml-washington/wk3/10.png)  

### noise
由于模型不可能完全反应客观世界，比如预测房价时不可能把邻居，卫生间数，车库等所有因素考虑进来，所以noise是必然存在的。
![quote](http://7xshuq.com2.z0.glb.clouddn.com/blog/img/figure/coursera/ml-washington/wk3/11.png)  

### bias
选择一个模型复杂度，如果使用的traing data不同，那么得到的模型也会有所不同。把所有可能的training data训练的到的模型取**平均**，并与理想的真实能反应客观的模型对比。
![quote](http://7xshuq.com2.z0.glb.clouddn.com/blog/img/figure/coursera/ml-washington/wk3/12.png)  
![quote](http://7xshuq.com2.z0.glb.clouddn.com/blog/img/figure/coursera/ml-washington/wk3/13.png)  
当模型复杂度低的时候bias高，模型复杂度高的时候bias低（虽然overfitting，但是由于取了平均，所以bias还是降低的）
![quote](http://7xshuq.com2.z0.glb.clouddn.com/blog/img/figure/coursera/ml-washington/wk3/14.png)  
![quote](http://7xshuq.com2.z0.glb.clouddn.com/blog/img/figure/coursera/ml-washington/wk3/19.png)  

### variance
如bias部分所述，固定复杂度，当得到的training data不同时训练得到的模型也有所不同。这些模型之间的差异就是variance。
![quote](http://7xshuq.com2.z0.glb.clouddn.com/blog/img/figure/coursera/ml-washington/wk3/15.png)  

当模型复杂度增加，由于overfitting，只要改变一两个样本点就会对模型产生很大的影响，于是variance就增加。
![quote](http://7xshuq.com2.z0.glb.clouddn.com/blog/img/figure/coursera/ml-washington/wk3/16.png)  
![quote](http://7xshuq.com2.z0.glb.clouddn.com/blog/img/figure/coursera/ml-washington/wk3/17.png)  
![quote](http://7xshuq.com2.z0.glb.clouddn.com/blog/img/figure/coursera/ml-washington/wk3/18.png)  

Again，bias和variance不能实际计算出来，而用MSE(Mean Square Error,即square error的期望值)来衡量总的error。
![quote](http://7xshuq.com2.z0.glb.clouddn.com/blog/img/figure/coursera/ml-washington/wk3/20.png)  

下图是随获得的数据集越大，training error和true error（即generalization error）的变化。注意在固定的模型复杂度下，数据集较少时traing error是较低的，但随着数据集越大，模型渐渐不能反应traing data，于是training error增大。随着数据集越来越大，training data实际是趋于与实际的population相等，于是两条曲线有共同的极限，该极限值是由于bias和noise造成的。
![quote](http://7xshuq.com2.z0.glb.clouddn.com/blog/img/figure/coursera/ml-washington/wk3/20.png)  
