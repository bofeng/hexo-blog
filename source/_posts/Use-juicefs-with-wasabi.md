---
title: Use juicefs with wasabi
typora-root-url: ../../source
date: 2022-09-13 10:28:01
tags: [juicefs]
---



Use [JuiceFS](https://www.juicefs.com/) we can mount a S3 storage to our local file system. Since JuiceFS is POSIX compatible, once mounted we can use common shell command to operate it.



## Installation

On linux:

```bash
curl -sSL https://d.juicefs.com/install | sh -
```

For other platforms, check here: https://www.juicefs.com/en/download/



## First-time setup

Here we will use [wasabi](https://wasabi.com/) as the object storage:

### Step 1: Format

```bash
#!/usr/bin/env bash
export ACCESS_KEY=<WASABI ACCESS KEY>
export SECRET_KEY=<WASABI SECRET KEY>
juicefs format --storage wasabi \
    --bucket https://yourbucket.s3.us-east-1.wasabisys.com \
    sqlite3:///home/juicefs/jfs.db jfs
```

Here we initialize juicefs with meta url `sqlite3:///home/juicefs/jfs.db`. 

### Step 2: Mount

```bash
juicefs mount sqlite3:///home/juicefs/jfs.db /jfs -d
```

This will mount our previous meta-url to local file system at path `/jfs`. Once mounted, you can `cd /jfs` to create some folders and files.

If you want to unmount it, just use:

```bahs
juicefs umount /jfs
```

Notice here the command is `umount`, not `unmount`, and the unmount parameter is not the meta-url, it is the mount point in your local file system.



## Restore an existing JuiceFS

You can dump and restore a file system to another machine. All you needs is restore JuiceFS meta info to the new meta url.

```bash
# 1, dump the meta json
juicefs dump sqlite3:///home/juicefs/jfs.db meta.json
```

Here we manually dump the meta info to a json file. If you already have the json file, for example, JuiceFS by default will save the meta info to a folder named "meta" in the object storage server (in this case, wasabi),every hour. If we go to wasabi, we will see a list of files under the bucket's meta folder, like this:

```
dump-2022-09-13-132813.json.gz
dump-2022-09-13-122226.json.gz
...
```

We can unzip this file to get the meta json:

```bash
gzip -d dump-2022-09-13-132813.json.gz
mv dump-2022-09-13-132813.json meta.json
```

Once we have the meta.json file, we can restore it to a new system. Since we are restoring the existing meta info, we don't need to format it here, only need to do the format when first initalize the file system. Also, please notice the "meta.json" file doesn't have our access key and secret key, so once loaded, we need to config those keys again:

```bash
# restore meta.json to meta-url engine
juicefs load sqlite3:///home/juicefs/jfs.db meta.json
juicefs config --access-key WASABI_ACCESS_KEY --secret-key WASABI_SECRET_KEY
juicefs mount sqlite3:///home/juicefs/jfs.db /jfs -d
```

Now if we go to `/jfs` , we will see our existing files.



## Mount at system reboot

We may want to auto mount JuiceFS when system (re)boot:

```bash
$ which juicefs
/usr/local/bin/juicefs
$ cp /usr/local/bin/juicefs /sbin/mount.juicefs
```

Now open `/etc/fstab`, add lines:

```
sqlite3:///home/juicefs/jfs.db    /jfs       juicefs     _netdev,max-uploads=50,writeback
```

The `max-uploads=50,writeback` are mount options, you can [check the full options here >>](https://juicefs.com/docs/community/command_reference#juicefs-mount)



## Performance test

Create one file or folder is very fast, for example, `mkdir abc` or `echo hello > hello.txt`, you barely can notice the time difference with operating in local drive. However, if you create many files in bulk, it may take a while.

For example, the command `virtualenv -p python3 venv3` will create a `venv3` folder that contains many files. This command can be done in 2 seconds in machine's local drive, but it may take while in JuiceFS mounted point, in my case, it took 58 seconds. But it is still better then [s3fs](https://github.com/s3fs-fuse/s3fs-fuse), which in my test cast, it took 5 minutes.



## Reference

* [How to Set Up Object Storage](https://juicefs.com/docs/community/how_to_setup_object_storage)

* [Mount JuiceFS at Boot Time](https://juicefs.com/docs/community/mount_juicefs_at_boot_time/)
* [JuiceFS Command Reference](https://juicefs.com/docs/community/command_reference)
