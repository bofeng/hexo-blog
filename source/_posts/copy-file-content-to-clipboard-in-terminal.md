---
title: 'Copy file content to clipboard in terminal'
date: 2020-04-19 19:45:32
tags: [linux,shell,ubuntu]
published: true
hideInList: false
feature: 
isTop: false
---
In Mac you can use `pbcopy`:
```bash
$ pbcopy < hello.txt
```

In Linux which is using X11 (I am using ubuntu), you can use `xsel` command:
```bash
$ sudo apt install xsel
$ xsel -b < hello.txt
```

To copy some program's output to clipboard, we can use pipe:
```bash
$ date | sel -b
```
Then if you use "ctrl-v", you will see the copied date.