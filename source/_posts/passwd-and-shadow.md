---
title: 'passwd & shadow'
date: 2019-04-07 02:59:11
tags: [linux,passwd]
published: true
hideInList: false
feature: 
---
Linux user's password is saved in `/etc/shadow` file, format like:
<!-- more -->


```bash
root:$6$MksUWINOmX.9ZXyP$yjO8RvJj5i9.G/mOx7ZA3npdX05iv5Z07k3zI/02LMBjPE01e8hUlVhMNNpzRIWG1n0n6flWZGgW2T/LsZGRT0:17885:0:99999:7:::
```

第一个字段是username, `$6` 是hash method，6表示SHA-512, `$MksUWINOmX.9ZXyP `是SALT。 `$yjO8RvJj5i9.G/mOx7ZA...WZGgW2T/LsZGRT0` 是加密后的字符串。

当用户使用`passwd`命令时，会将encrypted后的密码存入`/etc/shadow`文件。`/etc/shadow`文件是只有root可读可写的，那为什么普通用户也可以运行passwd命令写入此文件？这是因为passwd的"设置用户运行位"被set了。

```bash
$ ls -alh /usr/bin/passwd
-rwsr-xr-x 1 root root 59K Jan 25  2018 passwd
```

注意上面的rws中的s，表示不论是谁运行passwd，将运行的用户设置为此文件的拥有用户（也就是root），所以即使当普通用户运行passwd时，用`ps axu | grep passwd`也可以看到实际的运行用户是root。

Python验证登录：

```python
import crypt
print(crypt.crypt("myloginpwd", "$6$MksUWINOmX.9ZXyP"))
$6$MksUWINOmX.9ZXyP$yjO8RvJj5i9.G/mOx7ZA3npdX05iv5Z07k3zI/02LMBjPE01e8hUlVhMNNpzRIWG1n0n6flWZGgW2T/LsZGRT0
```

可以看到crypt后的字符串和shadow中的相同，说明密码(myloginpwd)是正确的。

