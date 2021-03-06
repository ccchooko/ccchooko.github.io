---
layout: post
title: "层次聚类"
date: 2016-11-13   
tag: 机器学习基础 
---


# 一、层次聚类原理介绍
***
### 1. Hierarchical clustering
1. 聚集法（Agglomerative）：这是一种“自下而上”的方法。将每一个原始数据分成一个cluster，计算这每个cluster之间的“距离”，然后将符合这个“距离”的cluster合并。执行若干次，最终把所有的数据聚到一个cluster中。而聚成什么样的结果，聚成几个类，跟自己设定这个“距离”的阈值而定。

2. 分割法（Divisive）：跟上面的方法恰好相反，是一种“自下而上”的方法。因为最终的结果跟上面的一样，但是程序实现的话聚集是比分割容易一些的，因此这个方法很少使用。

3.  基本步骤(聚集法)：
1). 将每个对象归为一类, 共得到N类, 每类仅包含一个对象. 类与类之间的距离就是它们所包含的对象之间的距离
2). 找到最接近的两个类并合并成一类, 于是总的类数少了一个.
3). 重新计算新的类与所有旧类之间的距离.
4). 重复第2步和第3步, 直到最后合并成一个类为止(此类包含了N个对象).

层次聚类树如下：
![](https://ws4.sinaimg.cn/large/006tNc79ly1fsuwv9wm6xj30bm099aal.jpg)

### 2. 皮尔逊积矩相关系数（Pearson product-moment correlation coefficient）：
（维基百科：https://en.wikipedia.org/wiki/Pearson_product-moment_correlation_coefficient）
在统计学中，它用于度量两个变量X和Y之间的相关（线性相关），其值介于-1与1之间。在自然科学领域中，该系数广泛用于度量两个变量之间的相关程度。
通常有以下公式计算：
1. 公式一：
![](https://ws2.sinaimg.cn/large/006tNc79ly1fsuwwpj2h7j30ef01f0sx.jpg)
2. 公式二：
![](https://ws2.sinaimg.cn/large/006tNc79ly1fsuwxj2jqgj308r01kdfw.jpg)
3. 公式三：
![](https://ws4.sinaimg.cn/large/006tNc79ly1fsuwxz6e8rj306501n74a.jpg)
4. 公式四：
![](https://ws1.sinaimg.cn/large/006tNc79ly1fsuwym8wwej308f02n0su.jpg)

> 通常情况下通过以下取值范围判断变量的相关强度：
相关系数     0.8-1.0     极强相关
                 0.6-0.8     强相关
                 0.4-0.6     中等程度相关
                 0.2-0.4     弱相关
                 0.0-0.2     极弱相关或无相关

以上的四个公式是等价的（为啥等价，有待研究），其中第一个公式比较有助于理解，就是用两个样本的协方差除以它们的标准差的积，用它来描述它们的相关性。下面程序实现的话，选择的是容易带入计算的公式四。


# 二、代码实现
***
### 1. 数据集
blogdata.txt
数据集选择的是网上已经整理好的数据，即从100篇英文blog中选出了1412个单词。部分数据如下：
![](https://ws1.sinaimg.cn/large/006tNc79ly1fsux0qo1bgj30ce07r75b.jpg)
 
其中第一列表示每篇blog名，每一行表示每篇blog的词频，把每一行的单词数目当作一个1412维向量进行处理。这样就是100个1412维的数据了，在通过上面介绍的皮尔逊系数去计算它们之间的相关性，并根据相关性把他们分成不同的类。

### 2. 源代码
clusterBase.py
> 这个是用来读取数据和计算皮尔逊距离的

```
from math import sqrt

def importData(FIFE='blogdata.txt'):
    blogwords = []
    blognames = []
    i = 0
    f = open(FIFE, 'r')
    for line in f:
        i += 1
        blog = line.split('\t')
        blognames.append(blog[0])
        if i != 1:
            blogwords.append([int(word_c) for word_c in blog[1:]])
    return blogwords, blognames

def pearson_distance(vector1, vector2):
    """
    Calculate distance between two vectors using pearson method
    See more : http://en.wikipedia.org/wiki/Pearson_product-moment_correlation_coefficient
    """
    sum1 = sum(vector1)
    sum2 = sum(vector2)
 
    sum1Sq = sum([pow(v, 2) for v in vector1])
    sum2Sq = sum([pow(v, 2) for v in vector2])
 
    pSum = sum([vector1[i] * vector2[i] for i in range(len(vector1))])
 
    num = pSum - (sum1 * sum2 / len(vector1))
    den = sqrt((sum1Sq - pow(sum1, 2) / len(vector1)) * (sum2Sq - pow(sum2, 2) / len(vector1)))
 
    if den == 0: return 0.0
    return 1.0 - num / den
```
hierachiclaCluster.py
> 主执行程序，产生聚类结果并且将结果画成图

```
# coding=utf-8
# /usr/bin/python
from clusterBase import importData, pearson_distance
# 将cluster封装成一个class，vec的类型是一个列表，表示的是每一篇blog的词频，left和right表示的是每个cluster左右相邻的cluster（这个是为
# 了后边画图用的，distance是两个vec之间的皮尔逊距离）
class bicluster:
    def __init__(self, vec, left=None, right=None, distance=0.0, id=None):
        self.left = left
        self.right = right
        self.vec = vec
        self.id = id
        self.distance = distance
 
def hcluster(blogwords, blognames):
    biclusters = [bicluster(vec=blogwords[i], id=i) for i in range(len(blogwords))]
    distances = {}  # 字典类型，key为(vec1,vec2),value为它们的皮尔逊距离
    flag = None;
    currentclusted = -1
    # 每次循环减少一个cluster，直到就剩一个cluster
    while (len(biclusters) > 1):
        min_val = 1;
        biclusters_len = len(biclusters)
        for i in range(biclusters_len - 1):
            for j in range(i + 1, biclusters_len):
                if distances.get((biclusters[i].id, biclusters[j].id)) == None:
                    distances[(biclusters[i].id, biclusters[j].id)] = pearson_distance(biclusters[i].vec,biclusters[j].vec)
                d = distances[(biclusters[i].id, biclusters[j].id)]
                if d < min_val:
                    min_val = d
                    flag = (i, j)
        bic1, bic2 = flag
        # 合并之后的vec为vec1和vec2的均值
        newvec = [(biclusters[bic1].vec[i] + biclusters[bic2].vec[i]) / 2 for i in range(len(biclusters[bic1].vec))]
        newbic = bicluster(newvec, left=biclusters[bic1], right=biclusters[bic2], distance=min_val, id=currentclusted)
        currentclusted -= 1
        del biclusters[bic2]
        del biclusters[bic1]
        biclusters.append(newbic)
    return biclusters[0]
 
'''
Print the tree structure, save as a jpeg image file.
'''
from PIL import Image, ImageDraw
 
def getheight(clust):
    if clust.left == None and clust.right == None: return 1
    return getheight(clust.left) + getheight(clust.right)
 
def getdepth(clust):
    if clust.left == None and clust.right == None: return 0
    return max(getdepth(clust.left), getdepth(clust.right)) + clust.distance
 
def drawdendrogram(clust, labels, jpeg='clusters.jpg'):
    h = getheight(clust) * 20
    w = 1200
    depth = getdepth(clust)
    scaling = float(w - 150) / depth
 
    img = Image.new('RGB', (w, h), (255, 255, 255))
    draw = ImageDraw.Draw(img)
    draw.line((0, h / 2, 10, h / 2), fill=(255, 0, 0))
 
    drawnode(draw, clust, 10, (h / 2), scaling, labels)
    img.save(jpeg, 'JPEG')
  
def drawnode(draw, clust, x, y, scaling, labels):
    if clust.id < 0:
        h1 = getheight(clust.left) * 20
        h2 = getheight(clust.right) * 20
        top = y - (h1 + h2) / 2
        bottom = y + (h1 + h2) / 2
        # line length
        ll = clust.distance * scaling
        draw.line((x, top + h1 / 2, x, bottom - h2 / 2), fill=(255, 0, 0))
        draw.line((x, top + h1 / 2, x + ll, top + h1 / 2), fill=(255, 0, 0))
        draw.line((x, bottom - h2 / 2, x + ll, bottom - h2 / 2), fill=(255, 0, 0))
        drawnode(draw, clust.left, x + ll, top + h1 / 2, scaling, labels)
        drawnode(draw, clust.right, x + ll, bottom - h2 / 2, scaling, labels)
    else:
        draw.text((x + 5, y - 7), labels[clust.id], (0, 0, 0))
 
if __name__ == '__main__':
    # pearson_distance
    blogwords, blognames = importData()
    clust = hcluster(blogwords, blognames)
    print clust
    drawdendrogram(clust, blognames, jpeg='blogclust1.jpg')
```              
### 3. 运行结果                     
![](https://ws4.sinaimg.cn/large/006tNc79ly1fsux1h7hroj30xc1j0q9d.jpg)
 
# 三、总结
采用层次聚类的方法效果看起来挺不错，但是时间复杂度太大。自底向上的聚集法的时间复杂度为$$O(n^{2}\log(n))$$，自顶向下的分割法的时间复杂度为$$O(2^{n})$$。层次聚类的算法运算量太大无法运用于大规模数据，所以一般是将层次聚类跟kmeans聚类结合起来使用。
用层次聚类算法得到信息的初始化信息，比如可以分成多少个簇，中心点的位置信息等，这样接下来可以利用这些信息来做Kmens聚类。这样的好处是可以解决Kmeans算法初始中心位置的随机性而且可以减少运算量，同时又避免了运用层次聚类算法的大运算量。