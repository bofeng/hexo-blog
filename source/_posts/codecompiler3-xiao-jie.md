---
title: 'CodeCompiler3小结'
date: 2018-10-15 14:05:11
tags: [python,linux,strace]
published: true
hideInList: false
feature: 
---
趁刚写完这个项目，把一些学到的东西写一下。
<!-- more -->


1, Python的`socketserver`是个写服务端TCP程序的神器，`ThreadingTCPServer`非常方便好用。

2, Python的`selectors`模块是多路复用的神器，`DefaultSelector`解决了好多问题。

3, 使用`selector`来监听`Popen`的进程的stdout和stderr时，需要把stdout和stderr用fcntl模块设置成非阻塞的：`fcntl.fcntl(proc.stdout, fcntl.F_SETFL, os.O_NONBLOCK)`

4, 自己启动的子进程，如果不是daemon mode，记得用`wait`函数或者`os.waitpid(pid, os.WNOHANG)`来回收。

5, `strace`用来分析程序运行时做的系统调用非常有帮助，`strace -p <pid> -f -e read` 用来监听某个pid进程的系统调用，可以把所有关于read的调用都打出来；`strace -o run.log -f python a.py` 可以跑某个程序(这个例子里是`python a.py`)，并把程序运行期间的系统调用都输入到run.log文件里。

6, 通过`strace`来监听程序有没有调用到`read(0, `可以对一般的程序来判断这个程序是不是要从标准输入stdin读取内容，也就是等待用户输入，但这个方法并不精确，理由有二：一是stdin严格来说用户是一直可以输入内容进去的，不需要非等到`read(0 `的系统调用，`read(0 `只是程序目前想从FD为0的标准输入stdin里阻塞式的读取数据，如果此时stdin里已经有了，那么程序就会直接读取，stdin里的内容完全可以是用户输入好的；二是，使用`dup`，`dup2`和`dup3`的系统函数，可以将其他打开的文件的FD指向stdin，例如这个文件的FD为4，在`dup2(4, 0)`的系统调用后，那么`read(4 `其实就是想从stdin里读取内容，所以从stdin里读内容不一定非要是`read(0 `可以是其他任意的FD数字，要看它之前是否有其他FD指向stdin。

7, 用`strace`分析了Golang的一个需要读取用户输入的程序，发现不管我输入多长，Golang的`read(0`系统调用，都是每次只读一个字符，比如我输入"hello\n"，Go要`read(0, buf, 1)`读取6次才能把这个字符串读完，也就是做了6次`read(0 `的系统调用，反观其他语言，比如C，是直接`read(0, buf, 1024)`，即一次调用会尝试读取1024个，虽然最后读到的是6个字符，但是只要一次read的系统调用就够了。Go为啥每次只读一个字符，匪夷所思。

8, Node.js来读取用户输入的时候，是使用的`read(12, `，原因就是它新开了其他的文件(FD为12)，并使用`dup3`把12指向了0，这样当`read(12, `时，其实就是从标准输入读取内容。

9, 使用strace来跟踪Java程序的系统调用时，一定要给strace使用`-f`选项，`-f`会跟踪所有启动进程/线程的系统调用，Java启动一个程序时会启动很多线程，如果不使用`-f`，strace输出的系统调用函数里你可能找不到想要的。

10, 大多数编程语言的stdout都有buffer，想要去掉这个buffer的话，对C语言来说可以加`stdbuf -o0`在前面，比如`stdbuf -o0 ./helloworld`；对Python来说，可以直接加一个`-u`的选项，比如`python3 -u hello.py`；对Ruby来说，可以先在/tmp下新建一个`rbopt.rb`的文件，内容为`$stdout.sync = true\n$stdin.sync = true`，然后使用`-r`选项让ruby在跑实际的ruby代码前先跑这个文件(相当于把stdout和stdin的buffer都去掉了)，例如`ruby -r /tmp/rbopt.rb hello.rb`。

11, Docker是个好东西，ENTRYPOINT可以直接指定跑一个程序(不能退出，也不能跑daemon模式)。如果一定要让自己的程序跑daemon模式，可以在跑之后再跑一个`tail -f /dev/null`，这样可以阻止container退出。

12, Python里想编译一个py文件到pyc文件，可以直接在命令行跑：`python3 -c "import py_compile; py_compile.compile('config.py', 'config.pyc')"`

13, `docker top <container id>`显出的进程id和`docker exec <container id> ps axu`的进程pid不同，前者显示的进程的pid是在host中的，后者显示的pid是在container中。比如你的程序在container里的pid可能是1，但是用`docker top`打出来的pid因为是在host机器上的，可能是一个非常大的四位数或者五位数。

14, 使用[Docker的Python](https://docker-py.readthedocs.io/)包来启动一个container时，可以指定`privileged`参数，让它等于`True`可以给这个container更高的系统权限，否则在Docker里跑的程序，即使是跑在root的用户下，也可能因为权限问题报错。

15, 前端使用的[jq-console包](https://github.com/replit-archive/jq-console)很好用(MIT License)，好评。