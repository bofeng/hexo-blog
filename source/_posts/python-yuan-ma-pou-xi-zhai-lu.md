---
title: 'Python源码剖析摘录'
date: 2014-05-18 21:00:00
tags: [python]
published: true
hideInList: false
feature: 
---
* PyObject {refcnt, type}
* PyVarObject {refcnt, type, ob_size}
<!-- more -->


在python中，我们只需要用一个PyObject*指针就可以引用任意的一个对象，而无论该对象实际是一个什么对象。

PyTypeObject中定义了大量的函数指针，这些函数指针最终都会指向一个函数，或者指向NULL。这些函数指针可以视为类型对象中所定义的操作，而这些操作直接决定着一个对象在运行时所表现的行为。比如PyTypeObject中的tp_hash指明对于该类型的对象，如何生成其hash值。

在一个对象的引用计数减为0时，与该对象对应的析构函数就会被调用，但是要特别注意的是，调用析构函数并不意味着最终一定会调用free释放内存空间，如果真是这样的话，那频繁地申请，释放内存空间会使python的执行效率大打折扣。

### Python对象从概念上大致分为5类：

* fundamental : type
* numeric : integer, float, boolean
* sequence: string, list, tuple
* dict : mapping
* internal : function, code, frame, module, method

### 检查加法结果是否溢出：

```c
x = a + b
if ((x^a) >=0 || (x^b) >= 0) { /*没有溢出*/ }
```

假设计算用4个bit表示整数，用补码能表示的范围就是-8到7 (1000是-8，替代了-0），举两个例子：

* -7 + -2 = -9 ：负数的补码为原码取反加1，所以-7的补码表示：1001， -2的补码表示：1110，相加为0111，所以 1001 ^ 0111 = 1110 小于0，1110 ^ 0111 = 1001也小于0，所以产生了溢出。
* 7 + 1 = 8 ：正数的补码与原码相同，0111 + 0001 = 1000，分别取异或，0111 ^ 1000 = 1111 小于0，0001 ^ 1000 = 1001 也小于0，所以产生了溢出。

Python小整数池默认存整数[-5,257)

Python的整数对象：先去小整数缓存池中找，如果有则返回；如果没有则在预先申请好内存的block上找一块free的内存初始化它。（涉及PyIntBlock {next, PyIntObject[]} 和PyIntObject* free_list）

### PyStringObject

Python的String对象，当创建了一个PyStringObject对象之后，该对象内部维护的字符串就不能再被改变了。

PyStringObject表示的字符串中间是可能出现\0，obsval指向的是一段长度为obsize+1个字节的内存。

PyStringObject中的obshash是缓存该对象的hash值，如果一个String对象没有被计算过hash值，那么obshash的初始值是-1。

Python始终会为字符串s创建PyStringObject对象，尽管s中维护的原生字符数组在interned中已经有一个与之对应的PyStringObject对象了，而intern机制是在s被创建后才起作用。通常Python在运行时创建了一个PyStringObject对象temp后，基本上都会调用 PyString_InternInPlace对temp进行处理，intern机制会减少temp的引用计数，temp对象会由于引用计数为0而被销毁，它只是会作为一个临时对象昙花一现。为什么要创建这么一个临时对象？因为interned这个PyDictObject的key是PyObject*。

String有单字符缓冲池(lazy init) size=255，在创建一个PyStringObject时，会首先检查所要创建的是否是一个字符对象，然后检查字符缓冲池中是否已经有了这个字符的字符对象的缓冲，如果有，则直接返回这个缓冲的对象即可。

string的join操作时，会首先统计出在list中一共有多少个PyStringObject对象，并统计这些PyStringObject对象所维护的字符串一共多长，然后申请内存，将list中的所有PyStringObject的字符串拷贝到新开辟的空间中去。如果使用+来连接字符串，则每次都会申请内存。

### PyListObject

是变长对象，也是可变对象（与PyStringObject）不同。

PyListObject结构体中有obsize和allocated，obsize相当于vector的size()，allocated相当于capacity()

PyObject** ob_item 表示存放的PyObject指针的数组。

Insert element to python array:
```c
for (i = n; --i >= where) {
    items[i+1] = item[i];
}
Py_INCREF(v);
item[where] = v;
```

remove list元素时，将会调用listassslice函数，在此函数中，当进行元素删除动作时，实际上通过memmove简单的搬移内存来实现。这就意味着，当调用list的remove操作删除list中的元素时，一定会触发内存搬移操作，这点跟C++的vector是完全一致的。

PyListObject对象缓冲池：free_lists，在删除PyListObject对象自身的时候，Python会缓冲池，查看其中缓冲的PyListObject的数量是否已经满了。如果没有，就将该待删除的PyListObject对象放到缓冲池中，以备后用。在以后创建新的PyListObject的时候，Python会优先使用在缓冲池中的PyListObject。

### PyDictObject

PyDictObject没有象map一样使用平衡二叉树，而是采用了hash table

当产生散列冲突时，Python会通过一个二次探测函数f，计算下一个候选位置addr，如果位置addr可用，则可将待插入元素放到位置addr；如果位置addr不可用，则Python会再次使用探测函数f，获得下一个候选位置，如此不断探测，总会找到一个可用的位置。

PyDictEntry {ssizet mehash; PyObject* key; PyObject* value}
一个Entry有三种状态，unused, active和dummy，其中dummy：当entry中存储的(key, value)对象被删除后，entry的状态不能直接从active转为unused，否则会导致冲突探测链的中断。相反，entry中的me_key将指向dummy对象，entry进入dummy态，这就是“伪删除”。

PyDictObject结构体中有一个ma_smalltable的PyDictEntry数组，这个数组默认大小是8，这也意味着当创建一个dict时，最少会有8个entry被创建。

当装载率大于或等于2/3时，就会改变table大小，当确定新的table大小时，通常选用的策略是新的table中的entry的数量是现在table中active态entry数量的4倍。当table中active态的entry非常大时（默认为50000），python只会要求2倍的空间。

PyDictObject的缓冲池：PyDictObject* free_dicts[80]，这个缓冲池机制和PyListObject中使用的缓冲池机制是一样的，开始时，这个缓冲池里什么也没有，直到第一个PyDictObject被销毁时，这个缓冲池才开始接纳被缓冲的PyDictObject对象。

和PyListObject中缓冲池机制一样，缓冲池只保留了PyDictObject对象，如果PyDictObject对象中ma_table维护的是从系统堆申请的内存空间，那么python将释放这块内存，归还给系统，而如果被销毁的PyDictObject中的table实际上并没有从系统堆申请，而是指向PyDictObject固有的ma_smalltable，那么只需要调整ma_smalltable中的对象的引用计数就可以了。
