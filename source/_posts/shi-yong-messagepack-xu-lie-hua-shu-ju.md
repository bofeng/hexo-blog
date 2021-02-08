---
title: '使用MessagePack序列化数据'
date: 2011-05-23 21:00:00
tags: [python]
published: true
hideInList: false
feature: 
---
[msgpack](http://msgpack.org/) 是一个序列化结构体的包。
<!-- more -->


通常web上做rpc通讯或者结构化数据存储，通常会采用json，但是缺点就是占用空间相比会比较大。google的protobuf是个不错的序列化替代方案。Tim在QConn介绍新浪微博的经验里说：

> web系统用json来存储及cache非常浪费。一条微博用json数据结构来存所有字段，包括作者信息，需要2~5k字节左右，用xml需要10k左右，用protobuf序列化后只有500字节。

虽然protobuf的压缩性能比较好，但是要实现定义message的结构体，不如json这么方便和容易更改。而msgpack相比之下就有这些优点：

* 兼容json的数据格式
* 比json的序列化更省时间和空间
* 支持很多种语言（python，java，ruby等等等等）

对python而言，需要先安装msgpack的包：

```bash
$ sudo easy_install msgpack-python
```

下面有个测试，序列化数据采用不同的方法，包括python的msgpack, simplejson和cPickle:

```python
import msgpack
import simplejson
import cPickle

msg = {"cmd":"update", "data":{"uid":123, "name":"vonbo"}}
pmsg = msgpack.dumps(msg)
print len(pmsg)

smsg = simplejson.dumps(msg)
print len(smsg)

cmsg = cPickle.dumps(msg)
print len(cmsg)
```

序列化之后的数据大小分别为34(msgpack)，56(simplejson)和87(cPickle)。

对于多次使用msgpack，可以先构造packer和unpacker对象：

```python
import msgpack

msg =  {"cmd":"update", "data":{"uid":123, "name":"vonbo"}}
packer = msgpack.Packer()
pmsg = packer.pack(msg)
unpacker = msgpack.Unpacker()
unpacker.feed(pmsg)
msg = unpacker.unpack()
```

然后用微博的一段稍微长些的json数据做测试，数据长度本身为76556字节（原数据有很多换行和空格）。

对这段数据分别使用msgpack, simplejson和cPickle：

* msgpack：dumps后长度2922字节，dumps 1000次耗时1.49秒，loads 1000次耗时0.49秒（这里使用上面用到的packer和unpacker）
* simplejson：dumps后长度4328字节，dumps 1000次耗时1.30秒，loads 1000次耗时3.30秒
* cPickle：dumps后长度5327字节，dumps 1000次耗时2.80秒，loads 1000次耗时1.91秒
* cPickle(2)：dumps后长度3682字节，dumps 1000次耗时1.60秒，loads 1000次耗时1.05秒

从上面的例子来看, dumps后的长度msgpack最小，dumps的时间上msgpack和simplejson相当，loads的时间msgpack最小，而且小其它两个几倍（当然这只能代表python版本，跟各语言的实现有关），折中来看，msgpack有相当不错的表现，至于稳定性或者其他方面需要在实际的生产环境中进一步测试。（至少现在msgpack的官网上还没有写谁在用它）

补：上面的测试时间当然和具体的例子有关，同事sly的一个例子表现上，原json有2068的长度，msgpack dumps后有1273，cPickle(2)压后只有971 。msgpack dumps 1000次耗时1.08秒，loads耗时0.36秒；cPickle(2) dumps耗时1.14秒，loads 0.56秒。（simplejson无论在空间和时间上都不如这两位）。虽然msgpack在这次的例子中压缩后的大小稍逊于cPickle，但是从dumps和loads的时间上还是胜出，而且msgpack又支持多语言（java等），这一点也比cPickle强，综合来说，msgpack还是比较不错的选择。
