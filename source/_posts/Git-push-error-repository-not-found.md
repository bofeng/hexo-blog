---
title: 'Git push error: repository not found'
typora-root-url: ../../source
date: 2022-05-09 23:03:13
tags: [git]
---



## Problem

1 minute ago, I can successfully `git push` my code to my github private repo. Next minute I tried to push another change, got this error:

```bash
remote: Repository not found.
fatal: repository 'https://github.com/xxx/yyy.git/' not found
```



## Solution

```bash
git remote rm origin
git remote add origin https://<personal access token>@github.com/xxx/yyy.git
git push --set-upstream origin main 
```

Then try `git status` and `git push`again.



## Reference:

* [Stackoverflow: repo not found](https://stackoverflow.com/questions/10116373/git-push-error-repository-not-found)
