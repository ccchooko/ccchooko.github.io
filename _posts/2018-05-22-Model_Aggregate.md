---
layout: post
title: "模型融合和特征构建"
date: 2018-05-22   
tag: 机器学习基础
---


## 常见的 Ensemble 方法有这么几种： 
### 1、Bagging：
使用训练数据的不同随机子集来训练每个 Base Model，最后进行每个 Base Model 权重相同的 Vote。也即 Random Forest 的原理。 
### 2、Boosting：
迭代地训练 Base Model，每次根据上一个迭代中预测错误的情况修改训练样本的权重。也即 Gradient Boosting，Adaboost 的原理。比 Bagging 效果好，但更容易 Overfit。

**Bagging，Boosting二者之间的区别**

（1）样本选择上： 

Bagging：训练集是在原始集中有放回选取的，从原始集中选出的各轮训练集之间是独立的。 

Boosting：每一轮的训练集不变，只是训练集中每个样例在分类器中的权重发生变化。而权值是根据上一轮的分类结果进行调整。 

（2）样例权重： 

Bagging：使用均匀取样，每个样例的权重相等 

Boosting：根据错误率不断调整样例的权值，错误率越大则权重越大。 

（3）预测函数： 

Bagging：所有预测函数的权重相等。 

Boosting：每个弱分类器都有相应的权重，对于分类误差小的分类器会有更大的权重。 

（4）并行计算： 

Bagging：各个预测函数可以并行生成 

Boosting：各个预测函数只能顺序生成，因为后一个模型参数需要前一轮模型的结果。
### 3、Blending：
用不相交的数据训练不同的 Base Model，将它们的输出取（加权）平均。实现简单，但对训练数据利用少了。 
### 4、Stacking：
![](https://ws3.sinaimg.cn/large/006tNbRwly1fucoeoe2x2j30l30l9ta1.jpg)
> 新特征的创建。将prob1~N列与原始数据组成新的特征向量（样本集列数+N）训练LV2模型 
![](https://ws2.sinaimg.cn/large/006tNbRwly1fucoes6zpoj30jk0f9dh8.jpg)
