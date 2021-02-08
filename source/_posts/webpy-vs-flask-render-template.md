---
title: 'Web.py vs Flask (render template)'
date: 2019-03-11 14:40:02
tags: [python,web,webpy,flask]
published: true
hideInList: false
feature: 
---
测试两个框架的效率。逻辑相同，都是map "/" to handler，然后这个handler里render template `index.html` (继承自`base.html`)，渲染模版都是用jinja2
<!-- more -->


### web.py

```
import web
from web.contrib import template

render = template.render_jinja("/templates", encoding="utf-8")

urls = (
    '/', 'index'
)

class index:
    def GET(self):
        return render.index()

app = web.application(urls, globals())
application = app.wsgifunc()
```

### Flask

```
from flask import Flask
from flask import render_template
app = Flask(__name__)

@app.route('/')
def index():
    return render_template("index.html")

application = app
```

### 测试

本次配置Nginx + uWSGI，然后用ab压测：`ab -c 100 -n 200 http://localhost:8080/`

结果：

* web.py: 550 requests / sec
* flask: 1200 requests / sec

这速度差一倍！web.py这么坑吗。。。