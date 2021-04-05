---
title: Build golang binaries for different system & arch
typora-root-url: ../../source
date: 2021-04-04 23:51:46
tags: [golang]
---

You can build runnable binaries for different Operating Systems and Arch even you are not in that OS and Arch.

### Makefile samples:

```makefile
all:
	# macbook m1
	GOOS=darwin GOARCH=arm64 go build -o build/darwin-arm64-main
	# rasperry pi
	GOOS=linux GOARCH=arm go build -o build/linux-arm-main
	# common linux server
	GOOS=linux GOARCH=amd64 go build -o build/linux-amd64-main
	# common windows 64bit
	GOOS=windows GOARCH=amd64 go build -o build/windows-amd64-main	
clean:
	rm build/*
```



### Reference

* [List of Supported GOOS and GOARCH](https://gist.github.com/asukakenji/f15ba7e588ac42795f421b48b8aede63)

