---
layout: 	post
title:		"Regularization(Ridged Regression) & Cross Validation"
subtitle:	"Note: ml regression week4"
header-img:	"img/post-bg-deeplearning.jpg"
date:		2016-04-15
author: 	"Borg"
catalog:	true
tags:
    - 数据分析
    - 回归分析
    - 笔记
---
# Regularization
上周已经讲了overfitting，那么当我们所有的数据量特别大的时候，即使有很多个特征模型也很难overfit。  
![quote](http://7xshuq.com2.z0.glb.clouddn.com/blog/img/figure/coursera/ml-washington/wk4/1.png)   

当模型overfit的时候，weights会变得很大，因此如果各个特征都很重要不可去除，可以通过减小weights来减弱overfitting的程度。那么如何衡量w是否过大？可以用w的绝对值相加或者用w的平方相加，注意不能直接把w相加，因为w可能为绝对值很大的负值和正值，这样会相互抵消。
![quote](http://7xshuq.com2.z0.glb.clouddn.com/blog/img/figure/coursera/ml-washington/wk4/2.png)   

## cost function
于是我们训练的时候可以重新定义cost function，通过优化新定义的cost function找到平衡RSS和weights的数量级的模型。
![quote](http://7xshuq.com2.z0.glb.clouddn.com/blog/img/figure/coursera/ml-washington/wk4/3.png)   

## lambda
lambda（不知道是不是这样打）的取值对weights的影响，当lambda越大，weights越来越小，相当于模型的复杂度越来越低，相应的bias越大，variance越小。
![quote](http://7xshuq.com2.z0.glb.clouddn.com/blog/img/figure/coursera/ml-washington/wk4/4.png)   
![quote](http://7xshuq.com2.z0.glb.clouddn.com/blog/img/figure/coursera/ml-washington/wk4/5.png)   

把cost function写成矩阵形式如下：
![quote](http://7xshuq.com2.z0.glb.clouddn.com/blog/img/figure/coursera/ml-washington/wk4/6.png)   

## gradient of ridged regression cost
和先前的一样的计算方法求导，得出gradient如下
![quote](http://7xshuq.com2.z0.glb.clouddn.com/blog/img/figure/coursera/ml-washington/wk4/7.png)   

## option1: closed-form
法一：直接让gradient等于0,求得最优解
![quote](http://7xshuq.com2.z0.glb.clouddn.com/blog/img/figure/coursera/ml-washington/wk4/8.png)   
最后的解如下，当lambda大于0时括号内的矩阵总是可逆
![quote](http://7xshuq.com2.z0.glb.clouddn.com/blog/img/figure/coursera/ml-washington/wk4/9.png)   

## option2: gradient descent
法二： 原理和没有regularization的回归模型一样。
![quote](http://7xshuq.com2.z0.glb.clouddn.com/blog/img/figure/coursera/ml-washington/wk4/10.png)   
在同一次descent中，还是一个一个feature地优化。
![quote](http://7xshuq.com2.z0.glb.clouddn.com/blog/img/figure/coursera/ml-washington/wk4/11.png)   
代码差不多如下：
![quote](http://7xshuq.com2.z0.glb.clouddn.com/blog/img/figure/coursera/ml-washington/wk4/12.png)   

## intercept
一般不regularize intercept，对于closed-form就如下
![quote](http://7xshuq.com2.z0.glb.clouddn.com/blog/img/figure/coursera/ml-washington/wk4/17.png)   
而对于gradient descent就是在计算partial derivative的时候判断下是不是constant feature，是的话用之前的方法计算。代码大致如下：
![quote](http://7xshuq.com2.z0.glb.clouddn.com/blog/img/figure/coursera/ml-washington/wk4/18.png)   
其实还有种方法，就是预处理原始的数据，使之具有值为0的期望值。

# Cross Validation
当数据量很少不够分出足够的validation data的时候可以使用cross validation，即把数据**随机**分成K份，拿出一份作为validation，这样循环把每一份都拿出来一次，最后取平均的validation error来衡量lambda参数（当然也可以用于优化其它参数如degree）。
![quote](http://7xshuq.com2.z0.glb.clouddn.com/blog/img/figure/coursera/ml-washington/wk4/13.png)   
![quote](http://7xshuq.com2.z0.glb.clouddn.com/blog/img/figure/coursera/ml-washington/wk4/14.png)   
![quote](http://7xshuq.com2.z0.glb.clouddn.com/blog/img/figure/coursera/ml-washington/wk4/15.png)   

不过该方法的缺点是计算量大。
![quote](http://7xshuq.com2.z0.glb.clouddn.com/blog/img/figure/coursera/ml-washington/wk4/16.png)   

# Code
编程时特别要注意使用corss validation之前需要把数据打乱，才能实现**随机**分配数据为K份。  
---
<pre><code>
train_valid_shuffled = graphlab.toolkits.cross_validation.shuffle(train_valid, random_seed=1)
</code></pre>
还有regularization用graphlab可以直接在create里设置lambda的值，即l2_penalty。
---
<pre><code>
model=graphlab.linear_regression.create(
	training_set,
	l2_penalty=l2_penalty,
	features=features_list,
	target=output_name,
	validation_set=None,
	verbose=False)
</pre></code>
特别注意下标，python的下标是从0开始，最后一位不包括。如range(0,5)实际是[0,1,2,3,4]。在冒号运算中也是如此，如data[3：9]实际9号元素不包括，和R，MATLAB不一样。所以在cross validation的时候要注意每一份开始和结束的位置。corss validation代码如下
---
<pre><code>
def k_fold_cross_validation(k, l2_penalty, data, output_name, features_list):
    errors=[]
    for i in xrange(k):
        start=n*i/k
        end=n*(i+1)/k-1
        validation_set=data[start:end+1]
        training_set=data[0:start].append(data[end+1:n])
        model=graphlab.linear_regression.create(
		training_set,
		l2_penalty=l2_penalty,
		features=features_list,
		target=output_name,
		validation_set=None,
		verbose=False)
        validation_error=sum((validation_set[output_name]-model.predict(validation_set))**2)/len(validation_set) 
        #注意此处不需要加+sum(model.get("coefficients")['value']**2)，题目要求是validation error
        #并且该式是在训练模型的时候用于优化带该式的cost fucntion以防止overfitting的，而现在是用validation进行评估
        #评估阶段不需要加上该式！
        errors.append(validation_error)                        
    validation_error=sum(errors)/k                              
    return validation_error,model
</code></pre>


