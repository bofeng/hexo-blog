---
title: Use reflex to hot reload go application in development
typora-root-url: ../../source
date: 2021-04-04 11:50:40
tags: [golang, reflex]
---

In last article I mentioned [air](https://github.com/cosmtrek/air) for hot reload when develop a go application, then I found this [reflex](https://github.com/cespare/reflex) tool which is much lightweight.



## Installation

```bash
$ go get github.com/cespare/reflex
```



## Run

```bash
$ reflex -s go run server.go
```

It will monitor all of your .go file and the .html templates file under the folder. Painless!



## Reference

* [Reddit Discussion](https://www.reddit.com/r/golang/comments/mim499/cosmtrekair_live_reload_for_go_apps/gt6ia8w?utm_source=share&utm_medium=web2x&context=3)
* [Reflex Github Repo](https://github.com/cespare/reflex)

