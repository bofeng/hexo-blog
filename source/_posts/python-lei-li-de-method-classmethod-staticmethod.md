---
title: 'Python类里的method, classmethod, staticmethod'
date: 2019-03-16 14:42:50
tags: [python,classmethod,staticmethod]
published: true
hideInList: false
feature: 
---
Class Example

```python
class MyClass:
    def method(self):
        return "instance method called", self
    
    @classmethod
    def classmethod(cls):
        return "class method called", cls
    
    @staticmethod
    def staticmethod():
        return "static method called"
```

* 普通的method属于instance，第一个参数self是instance自己，所以可以被instance调用；因为`self.__class__`指向class本身，所以此method内可以访问所有的instance变量和方法，也可以访问所属的class的变量和方法。
* classmethod属于class，第一个参数cls是class自己，所以可以被class调用。又因为instance本身的self，可以使用`self.__class__`访问到class本身，所以也可以被instance调用；因为有`cls`指针但没有`self`指针，所以此method内只能访问class的变量和方法，
* staticmethod属于class，虽然没有cls做参数，但可以被class调用，此时class相当于一个命名空间的作用。既然能被class调用，因为self里有`self.__class__`，所以也能被instance调用；因为没有`cls`和`self`指针，所以此method内既不能访问class的变量和方法，也不能访问instance的变量和方法。

例子：

```
obj = MyClass()
obj.method()
obj.classmethod()
obj.staticmethod()
# 以上三种均可调用

MyClass.classmethod()
MyClass.staticmethod()
# 以上两种均可调用
MyClass.method()  # 报错
```

![](/post-images/1571683492457.png)

在上图中，箭头代表指针或引用，有指针指向的代表能引用或访问到。

#### Key Takeaways

* Instance methods need a class instance and can across the instance through `self`
* Class methods don't need a class instance, They can't access the instance but they have access to the class itself via `cls`
* Static methods don't have access to `cls` or `self`. They work like regular functions but belong to the class's namespace
* Static and class methods communicate and (to a certain degree) enforce developer intent about class design. This can have definite maintenance benefits.

