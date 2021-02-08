---
title: 'Flask Essentials For Web Developer'
date: 2018-09-27 14:56:20
tags: [python,flask]
published: true
hideInList: false
feature: 
---
Python's web framework is quite similar to each other, thanks to the WSGI standard, so the first time when you learn a new Python web development framework, there are a few essential things you should know, and those will help you to get started fast.
<!-- more -->


## 1, Read parameter from Query String:
```python
from flask import request
searchword = request.args.get("key")
```

## 2, Read parameter from post body
```python
from flask import request
username = request.form.get("username")
```

## 3, Read uploading file content from form body
```python
from flask import request
f = request.files['the_file']
content = f.read()
print(f.content_type)
print(f.mimetype)
print(f.name)
print(f.filename)
```
File object reference: [Data Structures - Werkzeug Documentation (0.14)](http://werkzeug.pocoo.org/docs/0.14/datastructures/#werkzeug.datastructures.FileStorage)

## 4, Return a response
Directly return rendered string:
```python
def handler():
	page_content = render_template("home.html")
	return page_content
```
Return a pre-made response object
```python
from flask import make_response

def handler():
	page_content = render_template("home.html")
	resp = make_response(page_content, 200)
	resp.headers["x-key"] = "value"
	return resp
```

## 5, Get & Set Cookie:
Get Cookie:
```python
from flask import request

# get cookie example
def handler():
	username = request.cookies.get('username')
```
Set Cookie:
```python
from flask import make_response

def handler():
	resp = make_response(render_template("home.html"))
	resp.set_cookie('username', 'the username')
	return resp
```

## 6, Redirect
### 301: Move permanently
Use this one if one of url is permanently moved to another location
```python
from flask import redirect
def handler():
	return redirect("/some-url", 301)
```

### 302: Found
```python
from flask import redirect
def handler():
	return redirect("/some-url", 302)
```

### 303: See other (most common)
Use this in the scenario that after user posted some data, system wants to redirect this user to get another page url
```python
from flask import redirect
def handler():
	return redirect("/some-url", 303)
```


## 7, Return response with status code
```python
# use an extra returned value as status code
def handler():
	page_content = render_template('error.html')
	return page_content, 404
```

## 8, Return response with custom headers
```python
# use an extra returned value (second or third) as header
def handler():
	page_content = render_template("home.html")
	headers = {
		"x-extra-key": "value"
	}
	return page_content, headers
	# or
	return page_content, 200, headers
```

## 9, Sessions
Flask's built-in session will save session info into user's client side (into cookie) after encrypting it, so the app need to specify a secret_key used for encryption.
DO NOT give this key to any one! It should only be saved in the server side.
```python
from flask import Flask, session, redirect

app = Flask(__name__)

# Set the secret key to some random bytes. Keep this really secret!
# generate a good secret key:
# $ python -c 'import os; print(os.urandom(16))'
app.secret_key = b'some-secret-key-for-encrypting-session-data'

def handler():
	if 'username' not in session:
		return "sign-in required"
	session['username'] = "some username"
	return redirect("/home", 303)
```

## 10, Request context
```python
from flask import request

def handler():
	print(dir(request))

# here are some attributes might be commonly used
request.base_url
request.blueprint
request.endpoint
request.environ
request.full_path
request.headers
request.host
request.host_url
request.is_secure
request.is_xhr
request.method
request.mimetype
request.path
request.query_string
request.referrer
request.remote_addr
request.scheme
request.url
request.url_root
request.user_agent
```

Additionally, the `request.environ` is a dictionary, looks like this:

```json
{
	'wsgi.version': (1, 0), 
	'wsgi.url_scheme': 'http', 
	// ...
	'SERVER_SOFTWARE': 'Werkzeug/0.14.1', 
	'REQUEST_METHOD': 'GET', 
	'SCRIPT_NAME': '', 
	'PATH_INFO': '/', 
	'QUERY_STRING': '', 
	'REMOTE_ADDR': '127.0.0.1', 
	'REMOTE_PORT': 64636, 
	'SERVER_NAME': '127.0.0.1', 
	'SERVER_PORT': '3333', 
	'SERVER_PROTOCOL': 'HTTP/1.1', 
	'HTTP_HOST': '127.0.0.1:3333', 
	'HTTP_CONNECTION': 'keep-alive', 
	'HTTP_CACHE_CONTROL': 'max-age=0', 
	'HTTP_UPGRADE_INSECURE_REQUESTS': '1', 
	'HTTP_USER_AGENT': 'Mozilla/5.0 (Macintosh; Intel Mac OS X 10_14_0)',
	'HTTP_DNT': '1', 
	'HTTP_ACCEPT': 'text/html,image/apng,*/*;q=0.8', 
	'HTTP_ACCEPT_ENCODING': 'gzip, deflate, br', 
	'HTTP_ACCEPT_LANGUAGE': 'en-US,en;q=0.9,zh-CN;q=0.8,zh;', 
	'werkzeug.request': <Request '<http://127.0.0.1:3333/>' [GET]>
}
```

`request.headers` is also a dictionary, for example:
```json
{
	"Host": "127.0.0.1:3333",
	"Connection": "keep-alive",
	"Cache-Control": "max-age=0",
	"Upgrade-Insecure-Requests": "1",
	"User-Agent": "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_14_0) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/69.0.3497.100 Safari/537.36",
	"Dnt": "1",
	"Accept": "text/html,application/xhtml+xml,application/xml;",
	"Accept-Encoding": "gzip, deflate, br",
	"Accept-Language: en-US,en;q=0.9,zh-CN;q=0.8,zh;q=0.7,zh-TW;q=0.6"
}
```

## 11, Logging
```python
from flask import current_app

current_app.logger.debug('A value for debugging')
current_app.logger.warning('A warning occurred (%d apples)', 42)
current_app.logger.error('An error occurred')
```

## 12, Flask Internal Config Value
Configuration Handling - Flask 1.0.2 documentation

* ENV
* DEBUG
* SECRET_KEY
* SESSION_COOKIE_NAME
* SESSION_COOKIE_DOMAIN
* SESSION_COOKIE_PATH
* SESSION_COOKIE_HTTPONLY
* SESSION_COOKIE_SECURE
* PERMANENT_SESSION_LIFETIME: set cookie expire time
* TEMPLATES_AUTO_RELOAD

## 13, Jinja Autoescaping

By default, Flask's render_template will auto escape all variables in jinja templates, if you want to unescape a variable, here are 2 ways:

1, use `|safe`: 
```html
{{ myvariable|safe }}
```

2, use `autoescape` block:
```html
{% autoescape false %}
    <p>autoescaping is disabled here
    <p>{{ will_not_be_escaped }}</p>
{% endautoescape %}
```


