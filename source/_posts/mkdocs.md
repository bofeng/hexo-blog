---
title: 'mkdocs'
date: 2016-04-04 23:16:45
tags: [mkdocs]
published: true
hideInList: false
feature: 
---
mkdocs真是写文档的大杀器，配合github pages使用，炒鸡方便。

<!-- more -->


### 使用步骤

#### 1) 安装mkdocs

```bash
$ pip install mkdocs
```

#### 2) Setup github repo

在github上新建一个repo，比如```mkdocs-test```，在本地clone一下，然后使用```mkdocs new```初始化基本的目录结构：

```bash
$ git clone https://your-repo/mkdocs-test.git
$ cd mkdocs-test
$ mkdocs new test
$ mv test/* ./
$ rmdir test
$ ls
README.md  docs   mkdocs.yml
```

```mkdocs.yml``` 是配置文件，```docs``` 里面是放markdown格式文档的。

#### 3) 本机浏览

```bash
$ mkdocs serve
```

然后就可以在```http://127.0.0.1:8000```下查看页面了。都修改好后commit & push修改。（因为生成的所有html页面默认会在一个新生成site的文件夹里，这个site文件夹是不需要在repo里的，所以可以使用 ```echo "site/" > .gitignore```来忽略site目录）


#### 4) 部署到正式机

```bash
mkdocs gh-deploy --clean
```

因为repo是在github下的，这行命令可以首先生成所有html页面到site的文件夹下，然后将整站发布到你github pages上去：

```bash
$ mkdocs gh-deploy --clean
INFO -  Cleaning site directory
INFO -  Building documentation to directory: /Users/fengbo/project/git/mkdocs-test/site
INFO -  Copying '/Users/fengbo/mkdocs-test/site' to 'gh-pages' branch and pushing to GitHub.
INFO -  Your documentation should shortly be available at: http://bofeng.github.io/mkdocs-test
```

稍等几秒就可以在上面的链接地址上看到了整站了！

### 配置文件 mkdocs.yml

```mkdocs.yml``` 是配置文件，这里有些比较常用的配置：

```
site_name: Marshmallow
theme: readthedocs
google_analytics: ['UA-36723568-3', 'mkdocs.org']
pages:
- Home: 'index.md'
- User Guide:
    - 'Writing your docs': 'user-guide/writing-your-docs.md'
    - 'Styling your docs': 'user-guide/styling-your-docs.md'
- About:
    - 'License': 'about/license.md'
    - 'Release Notes': 'about/release-notes.md'
extra_css:
    - css/extra.css
    - css/second_extra.css
```

### Custom Domain

上面的网站是发布在 ```http://bofeng.github.io/mkdocs-test``` 上的，如果你有一个现有的域名，想用类似 ```docs.yourdomain.com/project``` 指向它，可以简单的配置下```Apache```或者```Nginx```：

Apache config (需要在mods-enabled下enable rewrite和proxy两个module):

```xml
<VirtualHost *:443>
...
RewriteEngine  on  
RewriteRule    "^/project(.*)$"  "http://bofeng.github.io/mkdocs-test$1"  [P]
ProxyPassReverse "/project" "http://bofeng.github.io/mkdocs-test"
...
</VirtualHost>
```

### Reference

* [Mkdocs](http://www.mkdocs.org/)
* [Mkdocs Configuration](http://www.mkdocs.org/user-guide/configuration/)
* [Mkdocs deploy](http://www.mkdocs.org/user-guide/deploying-your-docs/)
