---
title: '🌟 Starlette Quick Manual'
date: 2020-02-10 07:39:23
tags: []
published: true
hideInList: false
feature: 
isTop: false
---
[Starlette](https://www.starlette.io/) is an ASGI web framework for Python. 

## 1, Get value from Query String
This is commonly used when trying to get param value from GET request.

```python
async def handler(request):
    name = request.query_params.get("name")
```

## 2, Get value from Path
```python
# In url if you config:
Route("/post/{postid}", endpoint=post)

# In handler function:
async def post(request):
    postid = request.path_params.get("postid")
```

## 3, Get value from Body
This is commonly used when trying to get param value from a POST request.

```python
async def handler(request):
    inp = await request.form()
    name = inp.get("name")
```

## 4, Get uploaded file content
```python
async def handler(request):
    inp = await request.form()
    uploaded_file = inp["filename"]
    filename = uploaded_file.filename           # abc.png
    content_type = uploaded.content_type    # MIME type, e.g. image/png
    content = await uploaded_file.read()       # image content
```

## 5, Return a rendered page response
```python
from starlette.templating import Jinja2Templates
TMPL_ENGINE = Jinja2Templates(directory="templates")

async def handler(request):
    data = {
        "request": request,
        # ... other data
    }
    return TMPL_ENGINE.TemplateResponse("home.html", data)
```

## 6, Return a JSON response
```python
from starlette.responses import JSONResponse

async def handler(request):
    return JSONResponse({"name": "Bo"})
```

## 7, Return a customized response (status code and headers)
The above `TemplateResponse` and `JSONResponse` are subclass of the `Response` class. We can return a customized `Response`:
```python
import json
from starlette.responses import Response

async def handler(request):
    data = {
        "name": "Bo"
    }
    return Response(json.dumps(data), media_type="application/json")
```
`Response` takes `status_code`, `headers` and `media_type`, so if we want to change a response's status code, we can do:
```python
return Response(content, statu_code=404)
```

And customized headers:
```python
headers = {
	"x-extra-key": "value"
}
return Response(content, status_code=200, headers=headers)
```
Since `TemplateResponse` and `JSONResponse` are subclass of `Response`, these 3 parameters can be applied to them.
```python
return TMPL_ENGINE.TemplateResponse("home.html", data, headers=headers)
# And
return JSONResponse({"name": "Bo"}, status_code=200, headers=headers)
```

## 8, Redirect
```python
from starlette.responses import RedirectResponse

async handler(request):
    # Customize status_code: 
    #   301: permanent redirect 
    #   302: temporary redirect 
    #   303: see others
    #   307: temporary redirect (default)
    return RedirectResponse(url=url, status_code=303)
```

## 9, Get & Set Cookies
```python
# get cookie
async def handler(request):
    request.cookies.get('mycookie')

# set cookie
async def handler(request):
    resp = Response(content)
    # set_cookie(key, value, max_age=None, expires=None, path="/", domain=None, secure=False, httponly=False)
    resp.set_cookie("mycookie", "value")
    # delete cookie: delete_cookie(key, path='/', domain=None)
    resp.delete_cookie("myoldcookie")
    return resp
```

## 10, Use session
Starlette provides a cookie-based session middleware:
```python
from starlette.middleware import Middleware
from starlette.middleware.sessions import SessionMiddleware

middlewares = [
    Middleware(SessionMiddleware,
                secret_key=config.SESSION_CONFIG["secret_key"],
                max_age=config.SESSION_CONFIG["max_age"]),
]

app = Starlette(routes=routes, middleware=middlewares)

# Then in handler you can use it
async def handler(request):
    request.session["login_user"] = "bo"
```

Since this middleware used `json` to serialize and deserialize data, the data you saved in session need to be `json` serializiable. For example, if you save a `ObjectId` object (from MongoDB) in session, it won't know how to use json to serialize it, thus will raise exception and cannot save to cookie.

## 11, Request context

### 1) URL Object: `request.url`
* Get request full url: `url = str(request.url)`
* Get scheme: `request.url.scheme` (http, https, ws, wss)
* Get netloc: `request.url.netloc`, e.g.: example.com:8080
* Get path: `request.url.path`, e.g.: /search
* Get query string: `request.url.query`, e.g.: kw=hello
* Get hostname: `request.url.hostname`, e.g.: example.com
* Get port: `request.url.port`, e.g.: 8080
* If using secure scheme: `request.url.is_secure`, True is schme is `https` or `wss`

### 2) Headers: `request.headers`

```python
{
    'host': 'example.com:8080', 
    'connection': 'keep-alive', 
    'cache-control': 'max-age=0', 
    'sec-ch-ua': 'Google Chrome 80', 
    'dnt': '1', 
    'upgrade-insecure-requests': '1', 
    'user-agent': 'Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_3) ...',
    'sec-fetch-dest': 'document', 
    'accept': 'text/html,image/apng,*/*;q=0.8;v=b3;q=0.9', 
    'sec-origin-policy': '0', 
    'sec-fetch-site': 'none', 
    'sec-fetch-mode': 'navigate', 
    'sec-fetch-user': '?1', 
    'accept-encoding': 'gzip, deflate, br', 
    'accept-language': 'en-US,en;q=0.9,zh-CN;q=0.8,zh;q=0.7,zh-TW;q=0.6', 
    'cookie': 'session=eyJhZG1pbl91c2_KiQ...'
}
```

### 3) Client: `request.client`

* `request.client.host`: get client sock IP
* `request.client.port`: get client sock port

### 4) Method: `request.method`

* `request.method`: GET, POST, etc.

### 5) Get Data
* `await request.body()`: get raw data from body
* `await request.json()`: get passed data and parse it as JSON
* `await request.form()`: get posted data and pass it as dictionary

### 6) Scope: `request.scope`

```python
{
    'type': 'http', 
    'http_version': '1.1', 
    'server': ('127.0.0.1', 9092), 
    'client': ('127.0.0.1', 53102), 
    'scheme': 'https', 
    'method': 'GET', 
    'root_path': '', 
    'path': '/', 
    'raw_path': b'/', 
    'query_string': b'kw=hello', 
    'headers': [
        (b'host', b'example.com:8080'), 
        (b'connection', b'keep-alive'), 
        (b'cache-control', b'max-age=0'), 
        ...
    ], 
    'app': <starlette.applications.Starlette object at 0x1081bd650>, 
    'session': {'uid': '57ba03ea7333f72a25f837cf'}, 
    'router': <starlette.routing.Router object at 0x1081bd6d0>, 
    'endpoint': <class 'app.index.Index'>, 
    'path_params': {}
}
```

## 12, Put varaible in request & app scope

```python
app.state.dbconn = get_db_conn()
request.state.start_time = time.time()
# use app-scope state variable in a request
request.app.state.dbconn
```

## 13, Utility functions
### 1) Use `State` to wrap a dictionary
```python
from starlette.datastructures import State

data = {
    "name": "Bo"
}
print(data["name"])
# now wrap it with State function
wrapped = State(data)
# You can use the dot syntaxt, but can't use `wrapped["name"]` any more.
print(wrapped.name)
```

### 2) check if request is a ajax request
Depend on what client library you are using, the header's key and value might be different. Here is an example when use `$.ajax` function from jQuery.
```python
return request.headers.get("x-requested-with") == "XMLHttpRequest"
```

### 3) login_required wrapper function
```python
import functools
from starlette.endpoints import HTTPEndpoint
from starlette.responses import Response

def login_required(login_url="/signin"):
    def decorator(handler):
        @functools.wraps(handler)
        async def new_handler(obj, req, *args, **kwargs):
            user = req.session.get("login_user")
            if user is None:
                return seeother(login_url)
            return await handler(obj, req, *args, **kwargs)
        return new_handler
    return decorator

class MyAccount(HTTPEndpiont):
    @login_required()
    async def get(self, request):
        # some logic here
        content = "hello"
        return Response(content)
```


## 14, Exceptions

Handle exception and customize 403, 404, 503, 500 page:

```python
from starlette.exceptions import HTTPException

async def exc_handle_403(request, exc):
    return HTMLResponse("My 403 page", status_code=exc.status_code)

async def exc_handle_404(request, exc):
    return HTMLResponse("My 404 page", status_code=exc.status_code)

async def exc_handle_503(request, exc):
    return HTMLResponse("Failed, please try it later", status_code=exc.status_code)

# error is not exception, 500 is server side unexpected error, all other status code will be treated as Exception
async def err_handle_500(request, exc):
    import traceback
    Log.error(traceback.format_exc())
    return HTMLResponse("My 500 page", status_code=500)

# To add handler, we can add either status_code or Exception itself as key
exception_handlers = {
    403: exc_handle_403,
    404: exc_handle_404,
    503: exc_handle_503,
    500: err_handle_500,
    #HTTPException: exc_handle_500,
}

app = Starlette(routes=routes, exception_handlers=exception_handlers)
```

## 15, Background Task
### 1) Put some async task as background task
```python
import aiofiles
from starlette.background import BackgroundTask
from starlette.responses import Response

aiofiles_remove = aiofiles.os.wrap(os.remove)

async def del_file(fpath):
    await aiofiles_remove(fpath)

async def handler(request):
    content = ""
    fpath = "/tmp/tmpfile.txt"
    task = BackgroundTask(del_file, fpath=fpath)
    return Response(content, background=task)
```

### 2) Put multiple tasks as background task
```python
from starlette.background import BackgroundTasks

async def task1(name):
    pass

async def task2(email):
    pass

async def handler(request):
    tasks = BackgroundTasks()
    tasks.add_task(task1, name="John")
    tasks.add_task(task2, email="info@example.com")
    content = ""
    return Response(content, background=tasks)
```

## 16, Write middleware
There are 2 ways to write middleware:

### 1) Define `__call__` function:

```python
class MyMiddleware:
    def __init__(self, app):
        self.app = app

    async def __call__(self, scope, receive, send):
        # see above scope dictionary as reference
        headers = dict(scope["headers"])
        # do something
        # pass to next middleware
        return await self.app(scope, receive, send)
```

### 2) Use `BaseHTTPMiddleware`

```python
from starlette.middleware.base import BaseHTTPMiddleware

class CustomHeaderMiddleware(BaseHTTPMiddleware):
    async def dispatch(self, request, call_next):
        # do something before pass to next middleware
        response = await call_next(request)
        # do something after next middleware returned
        response.headers['X-Author'] = 'John'
        return response
```
