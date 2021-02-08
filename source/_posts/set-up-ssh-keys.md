---
title: 'Set up ssh keys'
date: 2016-01-07 11:21:24
tags: [ssh,linux]
published: true
hideInList: false
feature: 
---
### Steps

1, Create RSA key pair

```bash
ssh-keygen -t rsa
```

Answer the question and enter passphrase.

By default, the public key is now located in ```/home/user/.ssh/id_rsa.pub``` and the private key is now located in ```/home/user/.ssh/id_rsa```

2, Copy public key to server

```bash
cat ~/.ssh/id_rsa.pub | ssh user@123.45.56.78 "mkdir -p ~/.ssh && cat >>  ~/.ssh/authorized_keys"
```

3, Disable password for root login

```bash
vim /etc/ssh/sshd_config
```

Change the line ```PermitRootLogin yes``` to:

```
PermitRootLogin without-password
```

Save and quit, then reload ssh:

```bash
reload ssh
```

Now you can't login via ```ssh root@123.45.56.78``` by using root's password.

### Reference

1. [How to set up ssh keys](https://www.digitalocean.com/community/tutorials/how-to-set-up-ssh-keys--2)
