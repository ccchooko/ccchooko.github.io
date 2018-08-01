---
layout: post
title: "Python中关于yield的的理解"
date: 2018-01-31   
tag: Python 
---

> 参考廖雪峰老师的博客文章[《Python yield 使用浅析》](https://www.ibm.com/developerworks/cn/opensource/os-cn-python-yield/index.html)和自己的一些碰到的问题的解决和自己的理解

1. 通常的for...in...循环中，in后面是一个数组，这个数组就是一个可迭代对象，类似的还有链表，字符串，文件。它可以是mylist = [1, 2, 3]，也可以是mylist = [x*x for x in range(3)]。它的缺陷是所有数据都在内存中，如果有海量数据的话将会非常耗内存。
2. 生成器是可以迭代的，但只可以读取它一次。因为用的时候才生成。比如 mygenerator = (x*x for x in range(3))，注意这里用到了()，它就不是数组，而上面的例子是[]。

**以上是数组和生成器在python中区别的体现，**而获取generator的方法就是，for...in...或者调用.__next__()，Python2中对应的是next()。

廖老师的文章中对我理解yield帮助最大的是关于文件读取的这个例子。
例子如下：*如果直接对文件对象调用 read() 方法，会导致不可预测的内存占用。好的方法是利用固定长度的缓冲区来不断读取文件内容。通过 yield，我们不再需要编写读文件的迭代类，就可以轻松实现文件读取：*
```
def read_file(fpath): 
   BLOCK_SIZE = 1024 
   with open(fpath, 'rb') as f: 
       while True: 
           block = f.read(BLOCK_SIZE) 
           if block: 
               yield block 
           else: 
               return
```

也就是说，使用yield之后，下次循环他能够记住你运行到哪里，就直接从这个地方开始，从而避免了直接把所有数据写入内存中。

基于自己对yield的理解，现在有一个需求将一个数列，比如：112；134；153；67；88；91；101...，现在想得到每个数和它之前的数的和，将这个和作为该位置的新的数。下面是yield实现代码：
```
    def add(list_):
        a = list_[0]
        yield a
        i = 1
        while i<len(list_):
            b = list_[i]+b
            a = b
            yield b
            i += 1
```

用yield实现起来很完美。直接将这个generator转成list就够实现需求。
