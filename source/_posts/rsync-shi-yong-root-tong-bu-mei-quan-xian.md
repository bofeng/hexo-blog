---
title: 'rsync使用root同步没权限'
date: 2019-02-22 02:25:55
tags: [rsync,linux]
published: true
hideInList: false
feature: 
---
用root用户配合sshkey-gen生成了公私钥，把公钥cp到remote的机器上，但是rsync执行的时候一直说Permission Denied (public key)。后来发现是因为我的`/etc/ssh/sshd_config`里面配置是：

```
Port 32200
PermitRootLogin no
PasswordAuthentication no
PubkeyAuthentication yes
ChallengeResponseAuthentication no
```

把其中的PermitRootLogin改成yes，重启sshd `service ssh restart` 就行了。