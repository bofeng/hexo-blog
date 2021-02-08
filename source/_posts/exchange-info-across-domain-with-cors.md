---
title: 'Exchange Info Across Domain with CORS'
date: 2019-01-29 02:20:56
tags: [web,cors]
published: true
hideInList: false
feature: 
---
假设你拥有两个站点，a.com和b.com，用户在b.com登录之后，在访问a.com时，需要自动检测到用户已经在b.com登录了，进而a.com可以根据用户的登录状态显示不一样的信息。
<!-- more -->


一种方法是使用jsonp，另一种是使用CORS，两种都是访问非同源网站获取信息时的解决方案，这里介绍CORS。

当a.com的代码中有ajax访问b.com的接口时，浏览器自动会带上`Origin`这个header在HTTP请求里，当b.com接收到这个请求后，可以从header中读取这个`Origin`字段，判断是不是来自a.com，然后返回当前登录用户的信息。要实现这个需要做到三点：

1) b.com允许Origin是来自a.com的请求
2) b.com允许来自a.com的ajax请求带上b.com的cookie
3) a.com发送访问b.com的ajax请求时，告诉浏览器带上b.com的cookie

这三点中，首先是b.com中的代码：

```python
from flask import Flask, session, request, jsonify
from urllib.parse import urlparse


@app.route("/cors/userinfo")
def cors_userinfo():
    headers = dict(request.headers)
    origin = headers.get("Origin", "")
    if not origin:
        return "Origin field is missing"

    domain = urlparse(origin)
    if domain.hostname != "a.com" and \
            (not domain.hostname.endswith(".a.com")):
        return "Origin is not allowed"
    
    # check protocol is not necessary but highly recommended
    if domain.scheme != "https":
        return "HTTPS is required"

    resp_headers = {
        "Access-Control-Allow-Methods": "GET",        # limit methods to only GET
        "Access-Control-Allow-Origin": origin,        # for 1)
        "Access-Control-Allow-Credentials": "true",   # for 2)
        "Vary": "Origin",                             # avoid browser cache
    }
    uid = session.get("uid", None)
    data = {
        "uid": uid
    }
    return jsonify(data), 200, resp_headers
```

上面代码中，返回header中的`Access-Control-Allow-Origin`是为了第一点，`Access-Control-Allow-Credentials: true`是为了第二点。

第三点中，需要a.com在使用ajax访问b.com的API时设置：

```javascript
function call_cors_api() {
    $.ajax({
        url: "http://b.com/cors/userinfo",
        type: "GET",
        xhrFields: {
            withCredentials: true
        },
        data: {},
        success: function (response) {
            console.log(response);
        },
        error: function (xhr, status) {
            console.log(xhr, status);
        }
    });
}
```

其中的`xhrFields`中`withCredentials: true`是关键。

这样当在a.com中调用b.com接口时，浏览器就会带上b.com已登录用户的cookie，b.com进而会返回给a.com当前登录用户的uid。


#### Reference
* http://www.ruanyifeng.com/blog/2016/04/cors.html
* https://www.bedefended.com/papers/cors-security-guide