---
title: '提升protobuf-python的解包/封包性能'
date: 2011-11-16 21:00:00
tags: [python,protobuf]
published: true
hideInList: false
feature: 
---
今天一台跑数据中转的服务器cpu满了，这个服务器的主要任务是接收网络数据包，用protobuf解包，取出数据做xxoo的事后，封成一个新的包发给后端服务器。
<!-- more -->


这个中转服务器是python做的，cpu的主要耗时在对网络数据包频繁的解包封包。也就是protobuf里的ParseFromString（解包）和SerializeToString（封包）函数。因为这两个函数均是python实现的，所以效率比较低。

既然是cpu问题，想到的就是把这块改成调用c函数，直接python调用c实现的so。然后就去找有没有可以生成.so的包，搜到一个：[protocyt](http://evgenus.github.com/protocyt/) ，这个包是基于cython做的。

正当要试用的时候，同事来告诉了一个更快的方法：

1. 进protobuf源包的python目录：
2. cd protobuf-2.4.1/python/
3. 增加一个环境变量：`export PROTOCOL_BUFFERS_PYTHON_IMPLEMENTATION=cpp`
4. 重新build protobuf的python包：`python setup.py build`
5. 可以看到输出中多了一个_net_proto2___python.so ，这个也就是proto python的c实现
6. 然后安装这个so，`python setup.py install`

接下来，在你的处理脚本中也增加这个环境变量，例如之前跑python处理protobuf的脚本名称叫python a.py，现在在跑这行命令之前，同样先增加环境变量：`export PROTOCOL_BUFFERS_PYTHON_IMPLEMENTATION=cpp`，然后 `python a.py`

此时查看你的python a.py进程所加载的so，就多了一个_net_proto2___python.so，这表示现在的解包封包函数已经用了这个cpp版的实现了，效果很明显，大概可以提升5倍。以我们服务器为例，cpu的负载从60%降到了12%。

关爷说他们之前还用了psyco（http://psyco.sourceforge.net/）来提升效率，大概是2倍，所以两种方法结合可以提升效率大概10倍。psyco之前我们在hi系统中对便签的解析也用过，也是很方便，貌似只要两行代码就行了。可惜的是它不支持64位系统，只好罢了。

至于protocyt的表现就没再测试了，因为还依赖于cython和jinja2，比较麻烦，还是这种原始protobuf包里就支持的好用。
