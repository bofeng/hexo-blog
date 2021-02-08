---
title: 'rocket.chat server stopped on an update'
date: 2020-05-12 18:33:01
tags: [rocketchat,ubuntu,snap]
published: true
hideInList: false
feature: 
isTop: false
---
Today our self-hosted rocket.chat server suddenly stopped running, and it turns out I am not the only one [experiencing this problem](https://forums.rocket.chat/t/sudden-502-error/6914)

When I check the server, use the command to check status of rocketchat server and its mongo:
```bash
$ service snap.rocketchat-server.rocketchat-server status
# this shows mongodb cannot be connected
$ service snap.rocketchat-server.rocketchat-mongo status
# this shows mongdb is not running
```

So apparently there is something wrong with MongoDB. When I check the mongo start log with command:
```bash
journalctl -u snap.rocketchat-server.rocketchat-mongo
```

There are some errors about the versioning: 
> [initandlisten] WiredTiger error (-31802) [1589310297:583337][24694:0x7fbd44f65d00], txn-recover: unsupported WiredTiger file version: this build  only supports major/minor versions up to 1/0,  and the file is version 2/0: WT_ERROR: non-specific WiredTiger error

It turns out snap did an update to our rocket.chat server, and updated it to the new version, which now runnign mongodb version 3.6 and changed some database files. Now the database files cannot be run under mongo3.4 anymore (which rocket.server needs).

So I followed this method [here](https://forums.rocket.chat/t/suddenly-mongod-is-not-starting/4983/2):
1. Download version 3.6 in mongodb's website
2. cd to version 3.6 bin: `./mongod --dbpath /var/snap/rocketchat-server/common/`
3. in the same 3.6 bin folder, use mongo connect to the server: `./mongo`
4. now is the key part, run commnad: `db.adminCommand( { setFeatureCompatibilityVersion: “3.4” } )`
5. exit both above mongo and mongod
6. cd to `/snap/rocketchat-server/current/bin`
7. run `./mongod --repair --dbpath /var/snap/rocketchat-server/common/`
8. once it is done, try start rocketchat-mongo: `service snap.rocketchat-server.rocketchat-mongo start`, then check its status again: `service snap.rocketchat-server.rocketchat-mongo status`, it should be running now.

The MongoDB is running, but when I use:
```bash
service snap.rocketchat-server.rocketchat-server start
```
to start the rocket.chat server, it still doesn't work. By checking its log:
```bash
journalctl -u snap.rocketchat-server.rocketchat-server
```
Found there is a line: 
> fs.js:646
> return binding.open(pathModule._makeLong(path), stringToFlags(flags), mode);
>                 ^
> Error: ENOENT: no such file or directory, open '/snap/rocketchat-server/1436/star.json'

When I cd to `/snap/rocketchat-server/1436/` folder, there is no "star.json" file. But the old version 1427 has it: `/snap/rocketchat-server/1427/`

So I guess there is an error in this new version, have to roll it back to previous version and forbid it to update the latest version:
```bash
$ snap revert --revision=1427 rocketchat-server
$ snap run --shell rocketchat-server
$ snapctl get snap-refreshing #which returned "true"
$ snapctl set snap-refreshing=false
```
Now start the server again:
```bash
$ service snap.rocketchat-server.rocketchat-server start
```
check the status:
```bash
$ service snap.rocketchat-server.rocketchat-server status
```
Now it works!!! after spending several hours on this ... what a fun day :D

### Reference:
* [https://rocket.chat/docs/installation/manual-installation/ubuntu/snaps/](https://rocket.chat/docs/installation/manual-installation/ubuntu/snaps/)
* [https://forums.rocket.chat/t/snap-2-1-update-issues/5019](https://forums.rocket.chat/t/snap-2-1-update-issues/5019)
* [https://forums.rocket.chat/t/suddenly-mongod-is-not-starting/4983/2](https://forums.rocket.chat/t/suddenly-mongod-is-not-starting/4983/2)
* [https://forums.rocket.chat/t/sudden-502-error/6914/10](https://forums.rocket.chat/t/sudden-502-error/6914/10)


