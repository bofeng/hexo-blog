---
title: Dump and restore data from mongodb
typora-root-url: ../../source
date: 2021-08-06 22:33:01
tags: [mongodb]
---



## Download tools

MongoDB has separated its command line tools from the database binary, so to use `mongodump` and `mongorestore`, you need to [download the latest tools from their website here](https://www.mongodb.com/try/download/database-tools).



Once downloaded, unzip it, you will find the `bin` folder, inside which you could see the `mongodump` and `mongorestore`. Also `mongoimport` and `mongoexport`, etc.



## Dump

Here we use the `--archive` option to dump the db as a single archive file, named `db.archive`:

From a local database:

```bash
$ mongodump --uri=mongodb://<USER>:<PASSWORD>@localhost:27017/<DATABASE> --archive=db.archive
```

From a mongodb atlas:

```bash
$ mongodump --uri mongodb+srv://<USER>:<PASSWORD>@<ATLAS-URL>/<DATABASE> --archive=db.archive
```



## Restore

Restore to a local database:

```bash
$ mongorestore --uri="mongodb://<USER>:<PASSWORD>@localhost:27017/<DATABASE>" --archive=db.archive --nsExclude="admin.system.*"
```

Restore to mongodb atlas:

```bash
$ mongorestore --uri="mongodb+srv://<USER>:<PASSWORD>@<ATLAS-URL>/<DATABASE>" --archive=db.archive --nsExclude="admin.system.*"
```

