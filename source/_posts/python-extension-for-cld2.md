---
title: 'Python Extension for CLD2'
date: 2014-04-02 21:08:12
tags: [python]
published: true
hideInList: false
feature: 
---
有个Part-Time项目将来可能要用到这个语言检测模块，把编译步骤记下来。
<!-- more -->


CLD2:

下载项目源码 https://code.google.com/p/cld2/

下载[Python Extension zip file](/download/pycld2extension.zip)，解压后执行以下步骤：

1. 进public文件夹，新增cld_c_api.h
2. 进internal文件夹，新增cld_c_api.cc
3. 在internal文件夹内，修改compile_libs.sh : 在26行左右，-o libcld2.so之前，添加cld_c_api.cc，保存退出
4. 运行compile_libs.sh，成功后在internal文件夹内会有libcld2.so
5. 进public文件夹，把pycld2文件夹拖进去：里面包含三个文件Makefile, pycld2.c, test.py
6. 修改Makefile里的include和lib路径，保存退出后make，生成pycld2.so
7. 运行make test

目前的python API只有一个接口: detect_language，第一个参数是test str，第二个参数表示是否是plain text，默认是False，返回结果是个字典，包含lang_code, lang_name, is_reliable

```python
import pycld2
tests = "this is for testing"
print pycld2.detect_language(tests)
```

返回结果：

```javascript
{'lang_code': 0, 'is_reliable': True, 'lang_name': 'ENGLISH'}
```

下一步准备把它集成进项目里。
