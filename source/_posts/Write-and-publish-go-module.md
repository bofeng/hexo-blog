---
title: Write and publish go module
typora-root-url: ../../source
date: 2021-08-17 15:16:49
tags: ["golang"]
---



## Problem

You want to write and publish your own go module and release different versions.



## Solution

### V1

For the v1 version, in your `go.mod` file:

```go
module github.com/yourname/yourmodule
```

Tag it with v1.0.0:

```bash
$ git tag v1.0.0
$ git push --tags
```

Then in other projects, you can use `go get github.com/yourname/yourmodule` to install it.

If you want to update the module and it is fully compatible with the v1 version, after you made the change, you can just tag it with a higher version following the semver standard, say v1.0.1:

```bash
$ git tag v1.0.1
$ git push --tags
```

In other projects, you can just use `go get -u github.com/yourname/yourmodule` to install the new version. It will only install the updated same version, in this case, v1, if you have released v2 and tag it with v2.0.0, `go get -u` won't install the v2 version.



### V2

If you have some breaking changes and want to release version 2, then in your `go.mod` file, you need to change your module line to:

```go
module github.com/yourname/yourmodule/v2
```

and in your modules's other files, you need to change the import path to explicitly include the `v2`, like: `github.com/yourname/yourmodule/v2/config`. If you don't want to change this manually, a tool could help you auto do that: `go get github.com/marwan-at-work/mod/cmd/mod` then do `mod upgrade`.

Now you can just tag your version and release it:

```bash
$ git tag v2.0.0
$ git push --tags
```

In other projects where use this module, you need to explicitly install this version:

```bash
$ go get github.com/yourname/yourmodule/v2
```

and change your code's import path to explicitly include `v2`.



## Reference

* [Golang - publish go modules](https://blog.golang.org/publishing-go-modules)
* [Go module 如何发布 v2](https://blog.cyeam.com/go/2019/03/12/go-version)
* [Practical Go Lessons - Go Modules](https://www.practical-go-lessons.com/chap-17-go-modules)

