---
title: Update github credential for git clone/push in macOS
typora-root-url: ../../source
date: 2021-08-14 12:49:35
tags: [git, github]
---



## Problem

Github [removed password authentication](https://github.blog/2020-12-15-token-authentication-requirements-for-git-operations/) on August 13, 2021, if you are still using password auth, you will fail to do `git push`. 



## Solution

Change your saved password to "Personal Access Token". 

### Step 1

Go to Github's [personal access token page](https://github.com/settings/tokens), click "Generate new token", and check the "repo" group permissions. Once done, it will generate a token. Copy this token.

### Step 2

Open the "Keychain Access" app, search "github", you will see a "Internet password" item, double click it, you will the the details window:

![Screen Shot 2021-08-14 at 12.53.42 PM](https://cdn.jsdelivr.net/gh/bofeng/drive@main/uPic/2021-1628960177638-Q3zE3l.png)

Check the "Show password", then change it to the token you generated in the first step, click "Save changes". Now go back to your terminal, the `git push` should work.
