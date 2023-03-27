---
title: Bookmark folder path for shell
typora-root-url: ../../source
date: 2023-01-06 13:08:15
tags: [bash, shell, bookmark]
---



## Problem

You want to bookmark / create shortcut to some path, so you can cd to that folder quickly. For example, if you have a folder in `/home/johndoe/project/company/abc`, instead of `cd` to this long path, you can set a bookmark to this path, such that next time you can directly cd to that folder with like `goto abc`



## Solution

Use this bashmarks: https://github.com/bofeng/bashmarks 

Steps:

```bash
git clone https://github.com/bofeng/bashmarks.git
cd bashmarks
make install
echo "source ~/.local/bin/bashmarks.sh" >> ~/.zshrc # to .bashrc if using bash
source ~/.zshrc
```

Now you can the following command to bookmark folders:

```bash
# set bookmark
cd /home/johndoe/project/company/abc
st abc	# set the bookmark, you can use any name you want, doesn't have to be "abc"
cd ~
gt abc	# now you are in the abc folder
```

 

## Reference:

* https://github.com/huyng/bashmarks
