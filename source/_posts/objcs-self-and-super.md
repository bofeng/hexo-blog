---
title: 'objc’s self and super'
date: 2010-08-22 21:00:00
tags: [objc]
published: true
hideInList: false
feature: 
---
在objc中的类实现中经常看到这两个关键字”self”和”super”，以以前oop语言的经验，拿c++为例，self相当于this，super相当于调用父类的方法，这么看起来是很容易理解的。
<!-- more -->


以下面的代码为例：

```objc
@interface Person:NSObject {
    NSString*  name;
}
- (void) setName:(NSString*) yourName;
@end

@interface PersonMe:Person {
    NSUInteger age;
}
- (void) setAge:(NSUInteger) age;
- (void) setName:(NSString*) yourName andAge:(NSUInteger) age;
@end

@implementation PersonMe
- (void) setName:(NSString*) yourName andAge:(NSUInteger) age {
    [self setAge:age];
    [super setName:yourName];
}
@end

int main(int argc, char* argv[]) {
    NSAutoreleasePool* pool = [[NSAutoreleasePool alloc] init]
    PersonMe* me = [[PersonMe alloc] init];
    [me setName:@"asdf" andAge:18];
    [me release];
    [pool drain];
    return 0;
}
```

上面有简单的两个类，在子类PersonMe中调用了自己类中的setAge和父类中的setName，这些代码看起来很好理解，没什么问题。

然后我在`setName:andAge`的方法中加入两行：

```objc
NSLog(@"self ' class is %@", [self class]);
NSLog(@"super' class is %@", [super class]);
```

这样在调用时，会打出来这两个的class，先猜下吧，会打印出什么？
按照以前oop语言的经验，这里应该会输出：

```
self ' s class is PersonMe
super ' s class is Person
```

但是编译运行后，可以发现结果是：

```
self 's class is PersonMe
super ' s class is PersonMe
```

self的class和预想的一样，怎么super的class也是PersonMe？

### 真相

self 是类的隐藏的参数，指向当前当前调用方法的类，另一个隐藏参数是_cmd，代表当前类方法的selector。这里只关注这个self。super是个啥？super并不是隐藏的参数，它只是一个“编译器指示符”，它和self指向的是相同的消息接收者，拿上面的代码为例，不论是用[self setName]还是[super setName]，接收“setName”这个消息的接收者都是PersonMe* me这个对象。不同的是，super告诉编译器，当调用setName的方法时，要去调用父类的方法，而不是本类里的。

当使用self调用方法时，会从当前类的方法列表中开始找，如果没有，就从父类中再找；而当使用super时，则从父类的方法列表中开始找。然后调用父类的这个方法。

### One more step

这种机制到底底层是如何实现的？其实当调用类方法的时候，编译器会将方法调用转成一个C函数方法调用，apple的objcRuntimeRef上说：

> Sending Messages
>
> When it encounters a method invocation, the compiler might generate a call to any of several functions to perform the actual message dispatch, depending on the receiver, the return value, and the arguments. You can use these functions to dynamically invoke methods from your own plain C code, or to use argument forms not permitted by NSObject’s perform… methods. These functions are declared in /usr/include/objc/objc-runtime.h.
>
> * objc_msgSend sends a message with a simple return value to an instance of a class.
> * objc_msgSend_stret sends a message with a data-structure return value to an instance of a class.
> * objc_msgSendSuper sends a message with a simple return value to the superclass of an instance of a class.
> * objc_msgSendSuper_stret sends a message with a data-structure return value to the superclass of an instance of a class.

可以看到会转成调用上面4个方法中的一个，由于_stret系列的和没有_stret的那两个类似，先只关注objc_msgSend和objc_msgSendSuper两个方法。

当使用[self setName]调用时，会使用objc_msgSend的函数，先看下objc_msgSend的函数定义：

```objc
id objc_msgSend(id theReceiver, SEL theSelector, ...)
```

第一个参数是消息接收者，第二个参数是调用的具体类方法的selector，后面是selector方法的可变参数。我们先不管这个可变参数，以[self setName:]为例，编译器会替换成调用objc_msgSend的函数调用，其中theReceiver是self，theSelector是@selector(setName:)，这个selector是从当前self的class的方法列表开始找的setName，当找到后把对应的selector传递过去。

而当使用[super setName]调用时，会使用objc_msgSendSuper函数，看下objc_msgSendSuper的函数定义：

```objc
id objc_msgSendSuper(struct objc_super *super, SEL op, ...)
```

第一个参数是个objc_super的结构体，第二个参数还是类似上面的类方法的selector，先看下objc_super这个结构体是什么东西：

```objc
struct objc_super {
    id receiver;
    Class superClass;
};
```

可以看到这个结构体包含了两个成员，一个是receiver，这个类似上面objc_msgSend的第一个参数receiver，第二个成员是记录写super这个类的父类是什么，拿上面的代码为例，当编译器遇到PersonMe里setName:andAge方法里的[super setName:]时，开始做这几个事：

1，构建objc_super的结构体，此时这个结构体的第一个成员变量receiver就是PersonMe* me，和self相同。而第二个成员变量superClass就是指类Person，因为PersonMe的超类就是这个Person。

2，调用objc_msgSendSuper的方法，将这个结构体和setName的sel传递过去。函数里面在做的事情类似这样：从objc_super结构体指向的superClass的方法列表开始找setName的selector，找到后再以objc_super->receiver去调用这个selector，可能也会使用objc_msgSend这个函数，不过此时的第一个参数theReceiver就是objc_super->receiver，第二个参数是从objc_super->superClass中找到的selector

里面的调用机制大体就是这样了，以上面的分析，回过头来看开始的代码，当输出[self class]和[super class]时，是个怎样的过程。

当使用[self class]时，这时的self是PersonMe，在使用objc_msgSend时，第一个参数是receiver也就是self，也是PersonMe* me这个实例。第二个参数，要先找到class这个方法的selector，先从PersonMe这个类开始找，没有，然后到PersonMe的父类 Person中去找，也没有，再去Person的父类NSObject去找，一层一层向上找之后，在NSObject的类中发现这个class方法，而NSObject的这个class方法，就是返回receiver的类别，所以这里输出PersonMe。

当使用[super class]时，这时要转换成objc_msgSendSuper的方法。先构造objc_super的结构体吧，第一个成员变量就是self, 第二个成员变量是Person，然后要找class这个selector，先去superClass也就是Person中去找，没有，然后去Person的父类中去找，结果还是在NSObject中找到了。然后内部使用函数objc_msgSend(objc_super->receiver, @selector(class)) 去调用，此时已经和我们用[self class]调用时相同了，此时的receiver还是PersonMe* me，所以这里返回的也是PersonMe。

### Furthor more

在类的方法列表寻找一个方法时，还牵涉到一个概念类对象的isa指针和objc的meta-class概念，这里就不再详细介绍。
