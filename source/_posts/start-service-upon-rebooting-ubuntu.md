---
title: 'Start service upon rebooting ubuntu'
date: 2015-12-13 11:27:49
tags: [ubuntu]
published: true
hideInList: false
feature: 
---
因为Linode最近重启的有点频繁，所以当重启完成后，需要auto start一些service。之前一直没去了解过```/etc/```下面那一坨```rc.d```是怎么回事，这次查了下，留作记录（操作环境是Ubuntu 14.04 LTS）。
<!-- more -->


### 1. runlevel简介

运行```ls /etc | grep ^rc```，会出现几个rcX.d的文件：rc0.d, rc1.d, ... rc6.d, rc.local, rcS.d，其中rc0~6.d, rcS.d都是文件夹，
这些数字代表了runlevel。

可以把runlevel目前只当成个环境变量而已，系统启动时会设置这个环境变量，当系统在不同的runlevel时，就会读取相应的rcX.d下的脚本。
比如如果在level 3，那么就会读取```/etc/rc3.d```下的script。

有8个runlevels, 从0到6有7个，加上以S命名的那个rcS.d，一共有8个。这其中0，1 和 6 are reserved，Runlevel 0
is used to halt the system and 6 to reboot the system. Runlevel 1 is  used  to  bring
the system  back  down into single-user mode, after which the runlevel will be S (single-user mode).

乍一看来，系统启动时好像是从rc0.d下的脚本顺序执行到rc6.d下的脚本，但实际并不是这样，上面已经解释了，这只是系统在不同level或者不同status时，会去找对应的文件夹下的脚本去执行。

### 2. 切换runlevel

当系统切换runlevel时，如果在当前level的一些service在新的runlevel不存在，那么在这个新的runlevel下，这个service需要停止，只启动在新level下的service。

如果查看其中一个rcX.d文件夹下的文件，会看到很多以K或者S开头的脚本：

```bash
$ ls /etc/rc1.d
K09apache2   K20exim4      K20redis_6379     K77ntp  S30killprocs  S70pppd-dns
K19sendmail  K20memcached  K20ssh-host-keys  README  S70dns-clean  S90single
```

这些在切换level时会被用到，以K开头的service会被停止，以S开头的service会被启动。而这个过程是
通过脚本```/etc/init.d/rc ```来完成的，查看这个脚本内容：

```bash
CURLEVEL=""
for s in /etc/rc$runlevel.d/K*
do  
    # Extract order value from symlink
    level=${s#/etc/rc$runlevel.d/K}
    ...
    startup stop $SCRIPTS
done
...
CURLEVEL=$level
for s in /etc/rc$runlevel.d/S*
    ...
    startup $ACTION $SCRIPTS # Simply treat ACTION as start
done
...
```

可以看到这个脚本会读取以K开头的服务，并调用```startup stop```去停止它，而以S开头的服务，会
调用```startup start```去启动它。


### 3. 启动顺序

当系统启动时，会执行```/etc/init/rc-sysinit.conf```，这个脚本会调用```/etc/init.d/rc```并以runlevel=S做为参数，让系统运新rcS.d下的服务，当完成后，会调用```telinit```命令来跳回系统的default runlevel. 系统的default runlevel也是在```/etc/init/rc-sysinit.conf```设置的：

```bash
env DEFAULT_RUNLEVEL=2
```

可以看到默认的runlevel是2，也可以通过系统命令```runlevel```查看当前系统的runlevel：

```bash
$ runlevel
N 2
```

附关于telinit:

The primary command used to change run levels is ```telinit```. Get it? "Tell Init" to do something, like this:

```bash
telinit 2
```

```telinit``` takes one argument on the command line. As always, see the man page for full details. Normally the argument will be one of: 0,1,2,3,4,5,6, or the letter 'S'. As you may have guessed, the numbers correspond to the run level you wish to move to. Using the 'S', for single-user, is the same as the number 1, but don't do it; the 'S' runlevel is intended for use by the UserLinux (Debian)system.

A note of caution is warranted here. You can easily use the ```telinit``` command to reboot (run level 6), or shutdown (run level 0) the system, but it is not recommended. Certain programs need special processing for an orderly shutdown. Bypassing the expected shutdown sequence can have dire effects on your data. Older _Unix_ systems are especially sensitive to shutdown/bootup operations.


### 4. What's rc.local

The script /etc/rc.local is for use by the system administrator. __It is executed after all the normal system services are started, at the end of the process of switching to a multiuser runlevel.__ You might use it to start a custom service, for example a server that's installed in /usr/local. Most installations don't need /etc/rc.local, it's provided for the minority of cases where it's needed.

在Ubuntu系统里有两个rc.local，一个位于 ```/etc/rc.local```，另一个在```/etc/init.d/rc.local```，有两个存在是由于历史原因，
为了向前兼容。如果打开```/etc/init.d/rc.local```，可以看到注释：

```bash
# Short-Description: Run /etc/rc.local if it exist
```

可见```/etc/init.d/rc.local```只是为了运行```/etc/rc.local```里的代码，而默认的```/etc/rc.local```里只有一行exit 0。
如果想要加一些非常简单的启动服务启动运行脚本，可以直接编辑```/etc/rc.local```。


### 5. 增加服务启动脚本

做了前面的准备工作，终于要切入正题了：增加一个自己的服务启动脚本

__方法一：rc.local__

如果你的启动服务脚本很简单，根据上面对rc.local的解释，只需要把它放进```/etc/rc.local```即可，如果之前没有更改过```rc.local```，那么它只有一行```exit 0```，我们自己加一行：

```bash
echo "hello `whoami`, `date`" >> ~/teststart
exit 0
```

保存文件退出，下次系统启动时，会自动运行这个文件，在teststart里就可以看到输出了。

__由于rc.local会在rcX.d里其它服务执行后执行__，所以这里我们也可以启动一些以赖于其它服务先启动的脚本，
假设我们的脚本是统计apache2启动后启动了多少进程，那么直接用：

```bash
echo "[rc.local] `ps axu | grep apache | grep -v 'grep' | wc -l` apache process started @ `date`" > /tmp/teststart
exit 0
```

重启机器发现/tmp/teststart里的内容：

```bash
$ cat /tmp/teststart
[rc.local] 20 apache process started @ Mon Dec 14 16:24:54 EST 2015
```

能正确统计就是因为同在rc2.d下的apache会先于rc.local启动执行。


__方法二：update-rc.d__

第二种方法就是把启动脚本设置到rcX.d，首先在```/etc/init.d/```下写好脚本，例如 ```hello_service```：

```bash
#!/usr/bin/env bash
echo "hello from rc.d, `whoami`, `date`" >> ~/teststart
```

设置脚本可执行，然后使用```update-rc.d```讲它设置进rcX.d的目录：

```bash
$ sudo chmod +x hello_service
$ sudo update-rc.d hello_service defaults
Adding system startup for /etc/init.d/hello_service ...
   /etc/rc0.d/K20hello_service -> ../init.d/hello_service
   /etc/rc1.d/K20hello_service -> ../init.d/hello_service
   /etc/rc6.d/K20hello_service -> ../init.d/hello_service
   /etc/rc2.d/S20hello_service -> ../init.d/hello_service
   /etc/rc3.d/S20hello_service -> ../init.d/hello_service
   /etc/rc4.d/S20hello_service -> ../init.d/hello_service
   /etc/rc5.d/S20hello_service -> ../init.d/hello_service
```

可以看到这条命令给对应的rc0,1,6.d的目录下添加了K开头的脚本软链接，上面提到了以K开头的代表Stop，也就是当runlevel进入0,1,6时，会stop这个service；而给rc2,3,4,5.d里添加了S开头的脚本软链接，所以进入runlevel 2,3,4,5时会启动这个service，而2正是默认的runlevel。

但是等一下，如果以K和S开头的这些文件都指向我们的```hello_service```，而```hello_service```里又只有一行echo的命令，那么它是怎么区分执行start还是kill？

Debian手册里提到：'S' files invoke their program with the 'start' parameter, the 'K' files invoke their program with the 'stop' parameter.

所以系统进入runlevel 2的时候，相当于执行了```K20hello_service stop``` 和 ```S20hello_service start```，但问题是```hello_service```这个脚本目前完全不接受参数，不论是传stop或者start，都会执行echo，这显然不是我们想要的。

如何写一个符合这套标准的service启动脚本？

### 6. LSB Init Script

Debian手册：Write the running script by following LSB ([Linux Standard Base](https://wiki.debian.org/LSBInitScripts)) standard:

LSB-compliant init scripts need to:

* provide, at least, the following actions: __start__, __stop__, __restart__, __force-reload__, and __status__. All of those, except for status, are required by the Debian Policy
* return proper exit status codes.
* document runtime dependencies.


__第一：响应5个命令__

要响应start, stop, restart, force-reload, status5个命令，因为这个hello_service并不是真正的一直跑在bg的service，
它输出一行后就退出了，所以处理这些命令其实很简单：

```bash
case "$1" in
    start)
        echo "Hello from rc.d `whoami`, `date`" >> /tmp/teststart
        ;;
    stop)
        echo "stop hello_service"
        ;;
    restart)
        echo "Hello from rc.d `whoami`, `date`" >> /tmp/teststart
        ;;
    force-reload)
        echo "Hello from rc.d `whoami`, `date`" >> /tmp/teststart
        ;;
    status)
        # 因为输出一行就退出了，所以query status时应该是一直都没有running
        # 除非正在echo的时候，刚好执行了query status，但是这个时间很短，暂不考虑
        echo "hello_service is not running"
        exit 1
        ;;
    *)
        exit 2
        ;;
esac

exit 0
```

__第二：恰当的返回值__

处理两种情况：

1. 当程序成功运行，返回0，接收到了未知的命令，返回非0，象上面的例子exit 0和exit 2。
2. Query status时，如果service正在运行，返回0，如果没有在运行，返回非0，象上面的exit 1，具体见下面```--status-all```部分的解释。

上面的代码符合这两点要求，所以不需要增加代码。

hello_service只是在开机时简单的echo一行，并不是一个常驻的service，所以上面的代码处理这5个命令很简单。假设需要写一个常驻的service，那么在query status时，如何返回正确的status码？

目前在```/etc/init.d/```下的service常用的做法就是，当程序start时，将程序的pid保存在某个文件里，然后当执行status查询时，读取这个pidfile里的pid，然后查询有此pid的进程是否存在。因为这种做法非常普遍，所以LSB提供了这些shell function，只要导入这些函数就可以直接用了。

下面以查询pptpd进程是否在运行为例：

```bash
#!/usr/bin/env bash

# 导入status_of_proc 函数
. /lib/lsb/init-functions

# 根据pidfile查询此process的status，如果存在返回0，否则返回非0
status_of_proc -p /var/run/pptpd.pid "/usr/sbin/pptpd" "pptpd"

# 打印程序的返回值，如果在运行，返回是0，如果没有在运行，这次测试返回的结果是3
echo $?
```

(顺便提一下```/etc/init.d/pptpd```文件里做status操作时，少了上面的```-p```，所以查询运行状态经常是not running)

如果把```status_of_proc```结合进上面的脚本，处理status命令时，应该类似```/etc/init.d/apache2```的做法，调用```exit $?```：

```bash
....
. /lib/lsb/init-functions

....
    status)
        status_of_proc -p $PIDFILE "apache2" "$NAME"
        exit $?
        ;;
....
```

关于更多LSB init functions，请查看[init script functions](http://refspecs.linuxbase.org/LSB_3.1.0/LSB-Core-generic/LSB-Core-generic/iniscrptfunc.html)


__第三：执行顺序 及 依赖执行__

因为Ubuntu默认的runlevel是2，也就是系统启动时，进入level S执行完rcS.d下的服务后，会telinit 2，进入runlevel 2，执行rc2.d下的脚本，
此时rc2.d下的脚本有：

```bash
S20hello_service
S20pptpd
S20rsync
S20screen-cleanup
...
S91apache2
S99rc.local
...
```

__执行顺序__

 The 'nn' is a two digit number from 01-99; lower number programs are executed first. By this method, services that have a dependancy can be certain their precursor has ran. The K and S signify simply Kill, or Start.

 这些脚本的执行启动顺序也是按照字母排序来执行的，例如，可以看到apache2是在hello_service后执行启动的。
 __同时，请注意这里的S99rc.local，它是指向了 ```/etc/init.d/rc.local```，被分配到了S99这个比较大的数字，
 这也解释了前面的，为什么rc.local会在其它服务执行后再执行。__


__依赖执行__

回到之前的那个问题，如果hello_service要做的事是在apache启动后，统计启动了多少进程，那么就有了依赖执行的问题，
要求hello_service在apache2 service执行后再执行，这里有两个方法：

__1)：__ 由于脚本执行顺序是字母序，所以我们可以重命名```S20hello_service``` 改为类似
```S92hello_service```，这样的话，就会先执行S91的apache2，再执行S92hello_service。

```bash
$ update-rc.d -f hello_service remove
# 指定执行顺序为92
$ update-rc.d hello_service defaults 92
```

执行完后可以看到hello_service被命名成了```S92hello_service```，这样就排在了apache2的后面。


__2)：__ 另一种方法是告诉系统脚本的dependency，这也是LSB Init Script的第三个要求。
LSB Init Script有固定的格式要求，下面是一个模板：

```bash
### BEGIN INIT INFO
# Provides:          scriptname
# Required-Start:    $remote_fs $syslog $time
# Required-Stop:     $remote_fs $syslog $time
# Default-Start:     2 3 4 5
# Default-Stop:      0 1 6
# Short-Description: Start daemon at boot time
# Description:       Enable service provided by daemon.
### END INIT INFO
```

其中```Required-Start```就是启动依赖项，如果我们要统计apache启动后，启动了多少进程，把我们的hello_service内容改为：

```bash
### BEGIN INIT INFO
# Provides:          hello_service
# Required-Start:    apache2
# Required-Stop:     apache2
# Default-Start:     2 3 4 5
# Default-Stop:      0 1 6
# Short-Description: Stat apache2 process number at boot time
### END INIT INFO

case "$1" in
    start)
        echo "`ps axu | grep apache | grep -v "grep" | wc -l` apache process started @ `date`" >> /tmp/teststart
        ;;
    stop)
        echo "stop hello_service"
        ;;
    restart)
        echo "`ps axu | grep apache | grep -v "grep" | wc -l` apache process started @ `date`" >> /tmp/teststart
        ;;
    force-reload)
        echo "`ps axu | grep apache | grep -v "grep" | wc -l` apache process started @ `date`" >> /tmp/teststart
        ;;
    status)
        echo "hello_service is not running"
        exit 1
        ;;
    *)
        exit 2
        ;;
esac

exit 0
```

修改后使用```insserv```来添加hello_service的服务：

```bash
# 如果没有insserv的命令，请先apt-get install insserv
# 如果显示已安装insserv，但是还是不能在命令行调用，使用：
$ sudo ln -s /usr/lib/insserv/insserv /sbin/insserv
# 添加hello_service，insserv会重新排序
$ insserv hello_service
# 此时再ls
$ ls /etc/rc2.d/
S01dns-clean
S01pppd-dns
S02apache2
S03hello_service
...
S03sysstat
S04rc.local
...
```

可以看到生成的启动文件已经被重新排过序了，hello_service被排在了apache2的后面。rc.local仍然被放在最后。


### 7. 关于service --status-all

写到这里已经完成了如何写一个自己的开机启动服务，这里顺带提一下```service --status-all```，在[Linode: reboot survival guide](https://www.linode.com/docs/uptime/reboot-survival-guide) 和其它stackoverflow的一些提问里，提到了这个```service --status-all```的命令，用来查看当前运行的服务，原文类似：

"""Display which services are currently running, and which are listed to start on boot:

```bash
sudo service --status-all
```

运行显示结果类似：

```bash
$ service --status-all
 [ + ]  apache2
 [ + ]  atd
 [ ? ]  console-setup
 [ + ]  cron
 [ - ]  dbus
 [ ? ]  dns-clean
 [ + ]  friendly-recovery
 [ - ]  hello_service
 ....
```

Services preceded by a ```[+]``` are currently running, while those following a ```[-]``` are not, ```[?]``` means the services doesn't have a status command, so there's no way the service command can work out what's what."""

但是说显示```[+]```表示service正在运行其实是不精确的，```service --status-all```的执行时，
会向所有```/etc/init.d```下的service发送```status```的命令，例如```service apache2 status```，
如果查看```/usr/bin/service```的源码就会发现：

```bash
out=$(env -i LANG="$LANG" PATH="$PATH" TERM="$TERM" "$SERVICEDIR/$SERVICE" status 2>&1)
if [ "$?" = "0" -a -n "$out" ]; then
    #printf " %s %-60s %s\n" "[+]" "$SERVICE:" "running"
    echo " [ + ]  $SERVICE"
    continue
else
    echo " [ - ]  $SERVICE"
    continue
fi
```

如果```status```的返回值是0，就显示加号，否则就显示减号，这也是为什么上面LSB Init Script标准里的第二条要求：
必须返回恰当的status code。而这也表明，是否显示加号取决于具体应用/Service的这个启动脚本怎么写，是否符合这第二条要求。

比如```/etc/init.d/apache2```的脚本里：

```bash
status)
      status_of_proc -p $PIDFILE "apache2" "$NAME"
      exit $?
```

调用了```status_of_proc```，并将这条命令的返回值返回，($? is used to find the return value of the last executed command)，如果apache在运行，那么status_of_proc返回的就是0，就会执行下一行的exit 0，进而在```--status-all```时显示的就是加号，
这也说明apache的脚本在status下返回了proper exit code。

再看redis生成的脚本，redis默认跑在6379的端口下，Ubuntu apt-get 安装redis时系统会生成一个redis_6379的文件，查看它处理status的代码：

```bash
status)
    if [ ! -f $PIDFILE ]
    then
        echo 'Redis is not running'
    else
        echo "Redis is running"
    fi
    ;;
```

可以看到如果redis没有在运行，那么会显示not running，但是并没有接下来调用exit 1来返回非0的exit code，
也就是并没有严格按照LSB Init Script的标准来写，这样最后程序返回的还是0。如果redis本身并没有运行，
当用```service redis_6379 status```时会打出"not running"，
但是当用```service --status-all```查看时，会发现它前面却是显示有加号的。

Fix它也很简单，只要在not running那行后加上exit 1就好：

```bash
    ....
    if [ ! -f $PIDFILE ]
    then
        echo 'Redis is not running'
        exit 1
    ....
```

总而言之，使用```service --status-all```时返回的加号和减号并不能精确的反应service的运行状态，只有当service的启动脚本严格按照LSB Init Script标准来写时才是准确的。


### 8. 小结

想要开机启动时启动自己的服务：

* 如果执行很简单，只是一两行代码的事，那么别折腾了，直接扔进```/etc/rc.local```就好；
* 如果是个系统级的服务，不仅想开机执行，而且想提供类似 ```service my_service [start|stop|restart|status]```
等一系列操作，那么可以写一个启动脚本，放入```/etc/init.d/```下，确保它符合LSB Init Script的三个标准，最后用```update-rc.d```
或者```insserv```给一个执行优先级即可。

### Reference:

1. [Linode: reboot survival guide](https://www.linode.com/docs/uptime/reboot-survival-guide)
1. [Stackoverflow: rc.local](http://unix.stackexchange.com/questions/49626/purpose-and-typical-usage-of-etc-rc-local)
1. [Question mark in service --status-all](http://askubuntu.com/questions/446950/what-is-in-service-status-all)
1. [Ubuntu man page: runlevel](http://manpages.ubuntu.com/manpages/lucid/man7/runlevel.7.html)
1. [Debian: an intro to runlevels](https://www.debian-administration.org/article/212/An_introduction_to_run-levels)
1. [Debian: making scripts run at boot time with Debian](https://www.debian-administration.org/article/28/Making_scripts_run_at_boot_time_with_Debian)
1. [Ubuntu: upstart howto](https://help.ubuntu.com/community/UpstartHowto)
1. [Debian wiki: LSBInitScripts](https://wiki.debian.org/LSBInitScripts)
