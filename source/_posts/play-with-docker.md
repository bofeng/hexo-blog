---
title: 'Play w/ docker'
date: 2015-07-12 17:57:56
tags: [docker]
published: true
hideInList: false
feature: 
---
最近看到很多docker的消息，去docker.com上下载玩了一下。这里记录下命令，以备下次参考。

以在Docker中跑一个Python的web application为例：
<!-- more -->


### 下载与安装

去官网上找 安装步骤

装好后，试跑一下：

```bash
$ docker run ubuntu echo hello
```

会自动下载ubuntu的image，并且执行echo hello的shell命令

### VPS

运行：

```bash
$ docker run -t -i ubuntu /bin/bash
```

进入ubuntu的image，然后执行了/bin/bash的命令，相当于有了自己的vps，可以向平时操作ubuntu一样随意操作它，注意，如果你输入exit退出这个image，你对系统的改动并不会保存，比如你进入后安装了很多软件，新建了很多文件，然后直接exit了，下次你再使用上面的命令进入时，这些都会丢失。

### 保存Image

2个方法来保存这些改动。

方法一：当你做完改动后（例如安装了很多软件，跑起了Apache）等等，然后可以起另一个shell（也可以使用Ctrl+p & Ctrl + q来detach终端，使用docker ps找到container id后，再使用”docker attach containerid”的命令attach回去），使用命令：

```bash
$ docker ps
```

然后你会看到你的cotainer id，保存这个container id，使用：

```bash
$ docker commit <container id> yourname/reponame
```

其中输入containerid时不需要全部的字符串，只要它的前面3~4位即可；yourname是在docker.com上注册的username，reponame你可以自己随便取。例如：

```bash
$ docker commit ecda vonbo/webdev
```

commit成功后使用:

```
$ docker images
```

可以查看已经在image列表里了。

方法二，做一个你自己的image，下次直接使用。这里需要先写一个Dockerfile，Dockerfile里包含了一些规则来告诉docker如何build一个image。这里以build一个跑flask的python application为例：

新建Dockerfile，内容为：

```bash
FROM ubuntu:14.04
RUN apt-get update
RUN apt-get install -y wget tar dialog vim nano build-essential
RUN apt-get install -y python python-dev python-distribute python-pip
RUN pip install flask
RUN mkdir /home/root
WORKDIR /home/root
RUN wget http://download.0coder.com/app.txt
RUN mv app.txt app.py
ENTRYPOINT  ["python", "app.py", "&"]
```

其中app.txt的内容为：

```python
from flask import Flask
app = Flask(__name__)

@app.route("/")
def hello():
    return "Hello World!"

if __name__ == "__main__":
    app.run(host="0.0.0.0", port=5000)
```

只是简单的输入hello world，运行在5000端口上。

写好Dockerfile后，就可以build它了。

```bash
$ docker build vonbo/webdev .
```

记得输入最后的点，表示在当前目录找Dockerfile。

等它build完成后，使用：

```bash
$ docker images
```

就可以在image列表中看到了。

### 运行

以刚才build好的webdev为例，运行：

```bash
$ docker run -d -t -p 8080:5000 vonbo/webdev
```

-p 8080表示将host的8080端口映射到container的5000端口，这个5000就是前面在flask里写的那个端口号。输入后使用：

```bash
$ docker ps -l
```

可以看到这个在运行的container。然后使用你的host ip地址看下效果，访问：http://yourhostip:8080/，
你就可以看到Hello World的输入了。

### 保存

将你做好的image push到server中去，这样你其它的server也可以把它拉下来快速使用。push使用：

```bash
$ docker push vonbo/webdev
```

这样会把这个image push到public的repo中，你在其它任何机器都可以使用:

```bash
$ docker pull vonbo/webdev
```

来获得这个image，然后运行。

但很多时候，我们希望这些image是私有的，不让别人搜到或者使用，这时可以使用tag。首先去docker.com中新建一个private的repo，免费的帐号可以新建一个private的repo，比如这里命名为vonbo/private。

然后在你本地跑container的时候，将这个container commit到这个private的repo里去，并给它一个tag，以上面的webdev为例：

```bash
$ docker login
$ docker commit edca vonbo/private:webdev
$ docker push vonbo/private
```

注意，因为是private的repo，在做这个操作之前需要先执行登录操作。这样你就把这个container push到了私有的repo。

理论上tag的使用貌似是让你对同一个container | image做不同的版本，但是这里比较tricky的使用了tag来区分不同的container | image，例如你想另外做一个image跑redis，跑起来后可以把那个container使用同样的方法push到private里去。这么做原因是免费账户只提供一个private repo，这样可以把多个container都放到你这个private里去。

### 使用

你已经把image都push到了你private的repo中，现在假设你要在windows中部署测试，在安装docker后，可以使用：

```bash
$ docker pull vonbo/private:webdev
$ docker run -d -t -p 8080:5000 vonbo/private:webdev
```

然后访问http://192.168.59.103:8080 就可以看到结果。
（192.168.59.103是docker跑虚拟机的默认ip地址）你还可以跑多个端口映射：

```bash
$ docker run -d -t -p 8081:5000 vonbo/private:webdev
```

然后访问http://192.169.59.103:8081 也可以看到结果。使用：

```bash
$ docker ps
```

可以看到跑了两个container的instance。

假设还做了另外一个private的redis的image，可以使用同样的方法将它跑起来：

```bash
$ docker pull vonbo/private:redis
$ docker run -d vonbo/private:redis
```

### 参考

1. [ubuntu上安装docker](http://web2.0coder.com/archives/!https://docs.docker.com/installation/ubuntulinux/)
2. [windows上安装docker](http://web2.0coder.com/archives/!https://docs.docker.com/installation/windows/)
3. [Build your own registry](http://web2.0coder.com/archives/!https://blog.codecentric.de/en/2014/02/docker-registry-run-private-docker-image-repository/)
4. [Working with docker hub](http://web2.0coder.com/archives/!http://docs.docker.com/userguide/dockerrepos/)
5. [How to Containizer Python Web Application](http://web2.0coder.com/archives/!https://www.digitalocean.com/community/tutorials/docker-explained-how-to-containerize-python-web-applications)
6. [Dockerizing a Redis service](http://web2.0coder.com/archives/!https://docs.docker.com/examples/running_redis_service/)
