---
title: '为iOS编译CLD2库'
date: 2014-03-04 21:00:00
tags: [iOS]
published: true
hideInList: false
feature: 
---
CLD是Google Chrome中使用的语言检测模块，当你用chrome浏览某个外文网页的时候，你会遇到在页面顶部问你要不要翻译成你使用的语言。今天花了点时间把它编译成在iOS可以调用的库。
<!-- more -->


### CLD & CLD2

CLD现在有两个版本：

* https://github.com/mzsanford/cld
* https://code.google.com/p/cld2/

我今天编的是第二个，从项目介绍看还是活跃的项目，几天前我co下来，今天还能update到新的代码。

### 编译步骤：

1. 从页面 https://code.google.com/p/cld2/source/checkout co下代码，假设你这一步包含源码的文件夹是cld2-readonly。
2. 新建Xcode project，选OS X > Framework & Library > C/C++ Library 库类型选择static（静态库）
3. 新建完项目之后，给项目添加源文件，需要两个文件夹，internal和public。需要分别拷贝文件进这两个文件夹。先将cld2-readonly下的public的两个.h文件拷贝进public文件夹；在cld2-readonly下的internal文件夹下有个compile_libs.sh，将里面使用g++编译命令涉及到的.cc文件和对应的.h文件都拷贝进项目的internal文件夹，然后还要将下面几个拷贝进internal文件夹：
 * cld2tablesummary.h
 * integral_types.h
 * langspan.h
 * port.h
 * stringpiece.h
 * utf8prop_lettermarkscriptnum.h
 * utf8repl_lettermarklower.h
 * utf8scannot_lettermarkspecial.h
1. 进项目的settings > Building Settings > Base SDK 改为Lastest iOS（写这篇文章时是iOS 7.0)，Architecture为armv7和armv7s（这项会根据你改Base SDK而自动改变），然后下面Supported Platform选iphoneos和iphonesimulator（这项也会根据你改Base SDK而自动改变）
1. 进项目的settings > Building Settings，将其中的C++ Language Dialect项从GNU++ 11改成GNU++ 98，原因是编译时如果使用默认的C++11，会有很多Semantic issue，大都是数据的隐式转换问题，要fix这种问题，则需要手动把这些代码全改掉，从隐式转换改成显式转换，改成C++98后不需要更改源代码。
1. 文件已经准备好，可以开始编译了。选target为iOS Device，Command+B 这样可以编译出为iOS真机用的静态库，编译后会存在项目的Build/Products/Debug-iphoneos文件夹下，然后重命名为libcld2-device.a；将target改为simulator(不含64-bit的simulator)后，再使用Command+B编译，可以编译出为iOS Simulator 32-bit用的静态库，在Build/Products/Debug-iphonesimulator下，然后重命名为libcld2-sim32.a；将target改为simulator (64-bit)后，然后更改项目的settings > Building Settings > Architecture更改为including 64-bit的那项，再使用Command+B编译，可以编译出64-bit的静态库，也在Build/Products/Debug-iphonesimulator下，然后重命名为libcld2-sim64.a。（如果你在当前项目文件夹下找不到build的文件夹，可以去~/Library/Developer/Xcode/DerivedData/ 这里找一下）【update:使用Xcode 5.1 with iOS 7.1编译时，在Architecture中已经没有了include 64-bit项，只要选择默认的standard arch就可以了】
1. 将编译后的这三个.a文件合并成universal的lib，使用lipo命令：
```
lipo -create libcld2-sim32.a libcld2-sim64.a libcld2-device.a -output libcld2.a
```
这样就会生成libcld2.a的静态库文件。

截止到现在生成的库文件函数签名是C++的，也就是说，当iOS项目使用这个lib时，必须要使用Objective-C++ Compiler，而不能使用Objective-C的Compiler，这可能会对一些旧的项目带来麻烦，因为如果旧的项目已经在使用ObjC compiler了，要改成用ObjC++来编译，可能会改很多地方。所以我们需要在这个lib里导出需要的c函数接口。

### 添加C函数接口

可以在public/compact_lang_det.h里看到，它提供DetectLanguage的函数接口，以导出这个函数为例：

1, 在public下添加cld_c_api.h文件，然后将internal/generated_language.h的typedef Language定义整个拷贝进cld_c_api.h文件，这样在这个.h文件中就有了Language的定义，接着在下面声明C接口函数：

```c
#ifdef __cplusplus
extern "C" {
#endif
Language CDetectLanguage(
                      const char* buffer,
                      int buffer_length,
                      bool is_plain_text,
                      bool* is_reliable);
#ifdef __cplusplus
}
#endif
```

2, 然后在internal下添加cld_c_api.cc文件，添加代码：

```c
#include "../public/cld_c_api.h"
#include "../public/compact_lang_det.h"  // For Language

Language CDetectLanguage(
                     const char* buffer,
                     int buffer_length,
                     bool is_plain_text,
                     bool* is_reliable) {

    CLD2::Language lang = CLD2::DetectLanguage(buffer, buffer_length, is_plain_text, is_reliable);
    return (Language)lang;
}
```

这样就完成了对这个C函数接口的实现，其实就是个Proxy。

3, 添加完后重复上面的编译步骤的最后两步，这样生成的libcld2.a里就有了c函数CDetectLanguage的定义


### 使用

在新的iOS项目中或者旧的项目中使用时：

1. 给项目添加编译步骤中使用的internal文件夹及其下面的所有.h文件；public文件夹及其下面的所有.h文件
1. 给项目添加编译好的libcld2.a
1. 如果项目使用的是ObjC编译器，想使用上一步定义的C函数接口，而不是C++函数接口，那么在使用时需要include cld_c_api.h，并且在项目的settings > Build Phases > Link Binary with Libraries 下添加libc++.dylib，因为原库实现会调用C++的STL，需要链接C++标准库。
1. 这些都做好之后，就可以在你的.m文件中include cld_c_api.h，然后调用CDetectLanguage这个函数了。


### 参考

* [使用现有的C & C++ code编译iOS lib](http://inote.apptrek.net/2011/10/howto-compile-native-c-codes-to-a-library-for-ios-development-in-xcode-take-mosquitto-for-an-example/)
* [Create a universal lib for both simulator and device](http://jaym2503.blogspot.in/2013/01/how-to-make-universal-static-library.html)
