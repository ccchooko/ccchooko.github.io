---
layout: post
title: "线性回归"
date: 2016-11-20   
tag: 机器学习基础 
---


## 线性回归（Linear Regression）

在回归分析中，只包括一个自变量和一个因变量，且二者的关系可用一条直线近似表示，这种回归分析被称为一元线性回归分析。如果回归分析中包括两个或两个以上的自变量，且因变量和自变量之间是线性关系，则被称为多元线性回归分析。
回归方程如下:  
$$h_\theta(x)=\theta_0+\theta_1x_1+\theta_2x_2+...+\theta_nx_n$$
所谓的训练也就是利用已知的变量$x_1,x_2,...,x_n$去求$\theta_0,\theta_1,\theta_2,...,\theta_n$。所谓的预测也就是用另一堆变量（$x_1,x_2,...,x_n$）去求$h(\theta)$的值，即预测值。


**以下面的一元回归分析为例**
我们想预测特定房子的价值，预测依据是房屋面积。
我们有下面的数据集：
![](https://ws1.sinaimg.cn/large/006tNc79ly1fsuxiic8hjj305k04njrh.jpg)
这是一个一元回归模型，预测得到的回归方程为：
$y = 28.77659574x + 1771.80851064$


**方程如何得来：**
也就是对于$\theta$如何求解，是通过损失函数$$J(\theta)=\frac{1}{2}\sum_{i=1}^n {(h_\theta(x^{(i)})-y^{(i)})^2}$$
如何调整$\theta$使$J(\theta)$取得最小值有很多方法，其中有最小二乘法和梯度下降法等。


**最小二乘法：**
$$
\theta=(X^TX)^{-1}X^T\vec{y}$$
其中X表示训练特征组成的矩阵，结果表示成$y$向量。但是此方法要求$X$是列满秩的，而且求矩阵比较慢。
**梯度下降法：**
因为用最小二乘法求$\theta$存在局限性，因此采用下面的梯度下降法求解近似解$\theta$。
由$J(\theta)$公式知道求$\theta^T$转换成了求$J(\theta)$的极小值。
即：
$$
\theta_j:=\theta_j-\alpha{\frac{\partial}{\partial\theta_j}J(\theta)}
$$
其中$\alpha$为学习速率，当$\alpha$过大时，有可能越过最小值；当$\alpha$过小，则可能造成迭代次数过多，收敛速度过慢。


**下面是训练上述房屋信息的程序源码：**
```
# coding=utf-8
import matplotlib.pyplot as plt
import numpy as np
import pandas as pd
from sklearn import datasets, linear_model


# Function to get data
def get_data(file_name):
    data = pd.read_csv(file_name)  # here ,use pandas to read cvs file.
    X_parameter = []
    Y_parameter = []
    for single_square_feet, single_price_value in zip(data['square_feet'], data['price']):  # 遍历数据，
        X_parameter.append([float(single_square_feet)])  # 存储在相应的list列表中
        Y_parameter.append(float(single_price_value))
    return X_parameter, Y_parameter


def linear_model_main(X_parameters, Y_parameters, predict_value):
    # Create linear regression object
    regr = linear_model.LinearRegression()
    regr.fit(X_parameters, Y_parameters)  # train model
    predict_outcome = regr.predict(predict_value)
    predictions = {}
    predictions['intercept'] = regr.intercept_
    predictions['coefficient'] = regr.coef_
    predictions['predicted_value'] = predict_outcome
    return predictions


X, Y = get_data('linearTest.csv')
predictvalue = 700
result = linear_model_main(X, Y, predictvalue)
print "Intercept value ", result['intercept']
print "coefficient", result['coefficient']
print "Predicted value: ", result['predicted_value']


# Function to show the resutls of linear fit model
def show_linear_line(X_parameters, Y_parameters):
    # Create linear regression object
    regr = linear_model.LinearRegression()
    regr.fit(X_parameters, Y_parameters)
    plt.scatter(X_parameters, Y_parameters, color='blue')
    plt.plot(X_parameters, regr.predict(X_parameters), color='red', linewidth=4)
    plt.xticks(())
    plt.yticks(())
    plt.show()


show_linear_line(X, Y)
```

**运行结果：**
![](https://ws1.sinaimg.cn/large/006tNc79ly1fsuxiw71nbj30bu03dwep.jpg)

![](https://ws1.sinaimg.cn/large/006tNc79ly1fsuxj956rwj30i80fwmy0.jpg)