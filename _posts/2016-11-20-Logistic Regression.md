---
layout: post
title: "逻辑回归"
date: 2016-11-20   
tag: 机器学习基础 
---


## 逻辑回归（logic regression）
### 1. 理解
logistic回归就是一个线性分类模型，它与线性回归的不同点在于：为了将线性回归输出的很大范围的数，例如从负无穷到正无穷，压缩到0和1之间，这样的输出值表达为“可能性”。这样的话在某些情况会更加合理，也会更有说服力。
##### 举例说明一下
拿肿瘤的大小跟是否是恶性肿瘤的的例子来说。
![](https://ws2.sinaimg.cn/large/006tNc79ly1fsuxe7yjtdj30pq03xmxo.jpg)
根据线性回归模型我们只能预测连续的值，然而对于分类问题，我们需要输出 0 或 1，
我们可以预测：
    当 $h_\theta$ 大于等于 0.5 时， 预测 y=1。
    当 $h_\theta$ 小于 0.5 时， 预测 y=0 对于上图所示的数据， 这样的一个线性模型似乎能很好地完成分类任务。假使我们又观测到一个非常大尺寸的恶性肿瘤，将其作为实例加入到我们的训练集中来，这将使得我们获得一条新的直线.
![](https://ws2.sinaimg.cn/large/006tNc79ly1fsuxf3q0i0j30j205v3z1.jpg)
这时，再使用 0.5 作为阀值来预测肿瘤是良性还是恶性便不合适了。可以看出，线性回归模型，因为其预测的值可以超越[0,1]的范围，并不适合解决这样的问题。我们引入一个新的模型， 逻辑回归， 该模型的输出变量范围始终在 0 和 1 之间。 逻辑回归模型的假设是： 
$h_\theta(x)=g(\theta^TX)$
其中：
X 代表特征向量
g 代表逻辑函数（ logistic function）是一个常用的逻辑函数为 S 形函数（ Sigmoid function），公式为：

$g(z)=\frac{1}{1+e^{-z}}$
该函数的图像为：
![](https://ws2.sinaimg.cn/large/006tNc79ly1fsuxfjs04mj30ha07o74h.jpg)
合起来，我们得到逻辑回归模型的假设：
对模型的理解：$h_\theta(x)=\frac{1}{1+e^{-\theta^TX}}$
$h_\theta(x)$的作用是， 对于给定的输入变量， 根据选择的参数计算输出变量=1 的可能性，即$h_\theta(x)=P(y=1|x;\theta)$
### 2. 代码实现
数据集：
```
-0.017612   14.053064  0
-1.395634  4.662541   1
-0.752157  6.538620   0
-1.322371  7.152853   0
0.423363   11.054677  0
0.406704   7.067335   1
0.667394   12.741452  0
-2.460150  6.866805   1
0.569411   9.548755   0
-0.026632  10.427743  0
0.850433   6.920334   1
1.347183   13.175500  0
1.176813   3.167020   1
-1.781871  9.097953   0
-0.566606  5.749003   1
0.931635   1.589505   1
-0.024205  6.151823   1
-0.036453  2.690988   1
-0.196949  0.444165   1
1.014459   5.754399   1
1.985298   3.230619   1
-1.693453  -0.557540  1
...
```
代码实现：
```
import matplotlib.pyplot as plt
from numpy import *


def loadDataSet():
    dataMat = [];
    labelMat = []
    fr = open('testSet.txt')
    for line in fr.readlines():
        lineArr = line.strip().split()
        dataMat.append([1.0, float(lineArr[0]), float(lineArr[1])])
        labelMat.append(int(lineArr[2]))
    return dataMat, labelMat


def sigmoid(inX):
    return 1.0 / (1 + exp(-inX))


def gradAscent(dataMatIn, classLabels):
    dataMatrix = mat(dataMatIn)  # convert to NumPy matrix
    labelMat = mat(classLabels).transpose()  # convert to NumPy matrix

    m, n = shape(dataMatrix)
    alpha = 0.001
    maxCycles = 500
    weights = ones((n, 1))

    for k in range(maxCycles):  # heavy on matrix operations
        h = sigmoid(dataMatrix * weights)  # matrix mult
        error = (labelMat - h)  # vector subtraction
        weights = weights + alpha * dataMatrix.transpose() * error  # matrix mult
    return weights


def GetResult():
    dataMat, labelMat = loadDataSet()
    weights = gradAscent(dataMat, labelMat)
    print weights
    plotBestFit(weights)


def plotBestFit(weights):
    dataMat, labelMat = loadDataSet()
    dataArr = array(dataMat)
    n = shape(dataArr)[0]
    xcord1 = [];
    ycord1 = []
    xcord2 = [];
    ycord2 = []
    for i in range(n):
        if int(labelMat[i]) == 1:
            xcord1.append(dataArr[i, 1]);
            ycord1.append(dataArr[i, 2])
        else:
            xcord2.append(dataArr[i, 1]);
            ycord2.append(dataArr[i, 2])
    fig = plt.figure()
    ax = fig.add_subplot(111)
    ax.scatter(xcord1, ycord1, s=30, c='red', marker='s')
    ax.scatter(xcord2, ycord2, s=30, c='green')
    x = arange(-3.0, 3.0, 0.1)
    y = (-(float)(weights[0][0]) - (float)(weights[1][0]) * x) / (float)(weights[2][0])

    ax.plot(x, y)
    plt.xlabel('X1');
    plt.ylabel('X2');
    plt.show()


if __name__ == '__main__':
    GetResult()
```
### 3. 运行结果
结果，返回了特征值的回归系数。我们的数据集有两个特征值分别是x1，x2。我们又增设了了x0变量。得到的结果
![](https://ws4.sinaimg.cn/large/006tNc79ly1fsuxgdf6sjj30bs030wem.jpg)

我们得出x1和x2的关系（设x0=1），0=4.12414349+0.48007329*x1-0.6168482*x2
 
画出x1与x2的关系图
![](https://ws1.sinaimg.cn/large/006tNc79ly1fsuxgn658ej30i80fwdh4.jpg)
 

 
 
