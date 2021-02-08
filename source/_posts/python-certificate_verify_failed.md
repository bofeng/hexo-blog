---
title: 'Python: CERTIFICATE_VERIFY_FAILED'
date: 2019-03-10 14:39:16
tags: [python,ssl]
published: true
hideInList: false
feature: 
---
使用python的urllib的urlopen一个https的api时，经常遇到这个错误。解决方案：
<!-- more -->


首先检查web server端配置的证书是否正确。使用nginx + Let's Encrypt的证书时，certificate文件需要用fullchain的那个：

```
server {
    ssl on
    ssl_certificate /root/.acme.sh/abc.com/fullchain.cer;
    # 这里不要使用：
    # ssl_certificate /root/.acme.sh/abc.com/abc.com.cer;
}
```

如果这一步配置是正确的，但是仍然抛以上CERTIFICATE_VERIFY_FAILED异常，如果不想在python端手动指定证书文件，可以考虑忽略SSL证书验证：

方法1）在发送请求前，运行：
```
import ssl

ssl._create_default_https_context = ssl._create_unverified_context
```

方法2）使用certifi包：

```
pip install certifi
```

代码：

```
import certifi
resp = urlrq.urlopen('https://abc.com/api/xyz.json', cafile=certifi.where())
```


方法3）设置环境变量告诉python不要验证ssl证书：

```
export PYTHONHTTPSVERIFY=0 && python send_req.py
```

方法4）使用`requests`包：

```
pip install requests
```

代码：
```
import requests
requests.get('https://abc.com/api/xyz.json', verify=False)
```



#### Reference:

1. https://stackoverflow.com/questions/27835619/urllib-and-ssl-certificate-verify-failed-error
2. http://blog.pengyifan.com/how-to-fix-python-ssl-certificate_verify_failed/