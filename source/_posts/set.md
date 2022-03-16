---
title: Setup MongoDB ReplicaSet
typora-root-url: ../../source
date: 2022-03-15 20:02:31
tags: [mongodb]
---

## 1, Prepare mongodb

Create mongodb user:

```bash
useradd -m mongodb -s /bin/bash
cd /home/mongodb
```

Get mongodb binary: [Download page](https://www.mongodb.com/download-center/community/releases)

```
wget https://fastdl.mongodb.org/linux/mongodb-linux-x86_64-ubuntu2004-5.0.6.tgz
tar zxf mongodb-linux-x86_64-ubuntu2004-5.0.6.tgz
mv mongodb-linux-x86_64-ubuntu2004-5.0.6 mongodb-5.0.6
```



## 2, Setup config file

0, change to directory: `cd mongodb-5.0.6`

1, Create `keyfile`:

```bash
openssl rand -base64 756 > keyfile
chmod 400 keyfile
```

2, Create data folder: `mkdir data`

3, Add `50.conf`:

```
systemLog:
   destination: file
   path: "/home/mongodb/mongodb-5.0.6/mongod.log"
   logAppend: true
processManagement:
   fork: true
   pidFilePath: "/home/mongodb/mongodb-5.0.6/mongod.pid"
net:
   port: 40286
   bindIp: 127.0.0.1,<add private ip here>
security:
   keyFile: /home/mongodb/mongodb-5.0.6/keyfile
setParameter:
   enableLocalhostAuthBypass: false
replication:
   replSetName: dbrs0
storage:
   dbPath: "/home/mongodb/mongodb-5.0.6/data"
   journal:
      enabled: true
   engine:
      wiredTiger
   wiredTiger:
      engineConfig:
         cacheSizeGB: 4
```



## 3, increase number-of-file limits

```bash
vim /etc/security/limits.conf
mongodb soft nofile 64000
mongodb hard nofile 64000
```

let sudo read this limit, since we are using sudo to start mongodb in rc.local.

```bash
vim /etc/pam.d/sudo
# then add
session    required   pam_limits.so
```



## 4, Create mongodb admin user

1, Change 50.conf, comments out sections:

* security
* setParameter
* replication 

2, start mongod: `./bin/mongod -f 50.conf`

3, connect to MongoDB: `./bin/mongo --port 40286`

4, create super user:

```javascript
use admin;
db.createUser({user: "abc", pwd: "xyz", roles: [{ role: "root", db: "admin"}]});
```

5, kill mongod process

6, in `50.conf`, uncomment "security", "setParameter", "replication" sections

7, start mongod: `./bin/mongod -f 50.conf`

8, create `adminconn.sh`:

```bash
./bin/mongo --port 40286 -u abc -p xyz --authenticationDatabase admin
```



## 5, Add rs member's ip/host mapping

```bash
$ vim /etc/hosts
192.168.200.100 rs0m0.db.local
192.168.200.101 rs0m1.db.local
192.168.200.102 rs0m2.db.local
```

> Reminder: also need to add those hosts in application server.



## 6, Add ufw allow IP in all members

```bash
ufw allow from 192.168.200.100 to any port 40286 proto tcp
ufw allow from 192.168.200.101 to any port 40286 proto tcp
ufw allow from 192.168.200.102 to any port 40286 proto tcp
```

also add allow ip from application server (assume it is 192.168.200.200) :

```bash
ufw allow from 192.168.200.200 to any port 40286 proto tcp
```



## 7, Start MongoDB server on server reboot

Edit file: `/etc/rc.local` 

```bash
#!/usr/bin/env bash

sudo -H -u mongodb bash -c "/home/mongodb/mongodb-5.0.6/bin/mongod -f /home/mongodb/mongodb-5.0.6/50.conf"

exit 0
```



## 8, Initiate Replica Set

Connect to database then init the replica set:

```javascript
rs.initiate(
  {
    _id : "dbrs0",
    members: [
      { _id : 0, host : "rs0m0.db.local:40286"},
      { _id : 1, host : "rs0m1.db.local:40286"},
      { _id : 2, host : "rs0m2.db.local:40286"}
    ]
  }
);
// let members[0] has priority to be primary
cfg = rs.conf()
cfg.members[0].priority = 3;
cfg.members[1].priority = 2;
cfg.members[1].votes = 1;
cfg.members[2].priority = 1;
cfg.members[2].votes = 1;
rs.reconfig(cfg);
```

Reboot 3 db servers.
