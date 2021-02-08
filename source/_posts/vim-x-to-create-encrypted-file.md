---
title: 'Vim -x to create encrypted file'
date: 2018-10-19 02:17:37
tags: [vim,linux]
published: true
hideInList: false
feature: /post-images/vim-x-to-create-encrypted-file.svg
---
使用`vim -x filename.txt`可以创建加密文件。

<!-- more -->

例子：

```
$ vim -x test.txt
$
Enter encryption key: ***
Enter same key again: ***
```

输入密码再输入你要输入的内容，保存退出。用`cat`查看文件内容：
```
$ cat test.txt
VimCrypt~01!�g%
```

可以看到输入的内容被加密了，此时用`vim test.txt`再打开这个文件，会让输入密码。

上面用cat显示的内容里，可以看到有个明文的前缀`VimCrypt~01!`，后面跟的是加密的内容，其实这个前缀就是给vim自己用来判断是不是加密文件的，也就是说，如果一个文件的开头几个字符是 `VimCrypt~01!`，vim就会认为这是一个加密文件，会让你输入密码来显示文件内容。所以如果直接创建一个plain text文件，内容我们输入： `VimCrypt~01!abcd`，然后用vim打开这个文件，vim会首先提示要输入密码（注意创建这个文件的时候其实是没有密码的），随便输入一个密码后，vim会尝试用这个密码来解密原有的内容，这个例子里，它会认为后面跟的abcd是原文，并且会你输入的密码来解密显示这个内容，所以最后显示的内容是乱的；遇到这种情况，如果要显示原来的全文，应该在输入密码的步骤直接留空回车，这样vim就会正确的显示`VimCrypt~01!abcd`。

所以这里有个trick，如果有段文件你不想让其他人轻易的用vim来编辑，在你的文本内容最前面加上`VimCrypt~01!`，当其他人用vim打开时，会提示他输入密码，他如果认为这个文本内容加密了，可能在这一步就直接放弃了；又或者随便输入了什么当密码，这样显示的内容就是乱的；只有当他直接回车的时候，才能跳过这个假的加密步骤 ～ 好沙雕的操作。

