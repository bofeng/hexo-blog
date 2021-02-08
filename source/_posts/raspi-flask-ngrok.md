---
title: 'raspi + flask + ngrok'
date: 2019-03-18 14:47:58
tags: [raspi,flask,ngrok]
published: true
hideInList: false
feature: 
---
使用Flask在Raspberry Pi上写web application，然后使用ngrok反向代理，让此web application可以在公网访问。
<!-- more -->


1, Install virtualenv and flask:

```bash
apt install python3-pip
pip3 install virtualenv
virtualenv -p python3 venv3
source venv3/bin/activate
pip install flask
```

Edit `run.py`:

```python
from flask import Flask
app = Flask(__name__)


@app.route('/')
def hello():
    return "Hello World!"

if __name__ == '__main__':
    app.run()
```

Run: `python run.py`, now you should be able to see it is running on 127.0.0.1:5000

2, Run ngrok

Go to ngrok.com to register account, then download ngrok (Linux ARM) version.

![](/post-images/1571683788917.png)

unzip the downloaded zip file, then copy the code in step 3 and paste in console.

Lastly, run `./ngrok http 5000` to create a tunnel from your local 5000 to ngrok.com's subdomain, ngrok will give you a domain like http://07ab32bf.ngrok.io/ , this is the domain mapped to your local flask code.


