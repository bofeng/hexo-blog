---
title: 'Bloom Filter'
date: 2015-11-27 09:43:29
tags: [algorithm]
published: true
hideInList: false
feature: 
---
《数学之美》的23章布隆过滤器描述了基本原理，简单来说就是：用m个bit来映射n个元素是否存在，（例如用16亿个bit来映射1亿个黑名单email是否存在），每个元素被映射到k个不同的位置，然后把这k个位置的bit置1。

<!-- more -->


伪代码如下：
```python
# Bloom filter pseudo code
# initialize all m bits to 0
bitVector = [0] * m

def add(ele):
  # get 8 integers
  position = [hash_func(ele) for hash_func in [func1, func2, ... func8]]
  for pos in position:
    bitVector[pos] = 1
    
def exist(ele):
  position = [hash_func(ele) for hash_func in [func1, func2, ... func8]]
  return all([bitVector[pos] == 1 for pos in position])
```

布隆过滤器不会漏掉任何已经在这个vector中的元素，但是它有极小的可能将一个不在其中的元素也判定为在其中（当两个元素被映射到的k个位置相同时），这种被称为False Positive。

当m/n和k的值比较大时，这种误判率越小，可以[参考这篇误识别率表](http://pages.cs.wisc.edu/~cao/papers/summary-cache/node8.html)。

> 布隆过滤器背后的数学原理在于两个完全随机的数字冲突概率很小，因此，可以在很小的误识别率条件下，用很少的空间存储大量的信息。常见的补救方法是建立一个小的白名单，存储那些可能被误判的信息。

其它参考：

1. [布隆过滤器详解（含误判率分析）](http://www.cnblogs.com/allensun/archive/2011/02/16/1956532.html)
2. Python版本实现：[python-bloomfilter](https://github.com/jaybaird/python-bloomfilter) and [pybloomfiltermmap]( https://github.com/axiak/pybloomfiltermmap)
