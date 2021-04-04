---
title: Install golang air tool in Macbook M1
typora-root-url: ../../source
date: 2021-04-03 20:56:38
tags: [golang, air, macos]
---

## Installation

Tried to install this "Live Reload Tool" [air](https://github.com/cosmtrek/air) in MacOS. In the README page, it listed 2 ways to install:

1, Use the `go get` way: `go get -u github.com/cosmtrek/air` , but this will only install an old version v1.12, which doesn't support auto read `.air.toml` as its default config. If you already created a `.air.toml` in your project and want to just run `air` inside that folder w/o explicitly specify the config file path, then you need a newer version

2, It also listed the bash way:

```bash
curl -sSfL https://raw.githubusercontent.com/cosmtrek/air/master/install.sh | sh -s -- -b $(go env GOPATH)/bin
```

But this doesn't work for Macbook M1 chip, b/c at the day of writing this post, it doesn't have a OS=darwin, Arch=arm64 in its [release page](https://github.com/cosmtrek/air/releases). The script tries to fetch `air_1.15.1_darwin_arm64.tar.gz` but it isn't there.



So we need to build it manually:

1. Clone the git: `git clone https://github.com/cosmtrek/air.git`
2. Check to a released version: `git checkout v1.15.1`
3. Run `make build`, this will generate binary file `air`
4. Copy it to your go bin path: `cp air $HOME/go/bin` (make sure go bin path is in your $PATH env)
5. Run `air version` to confirm it has been successfully installed: `air version` (should print v.1.15.1)



## Config file

In your project folder, the `.air.toml` may look like this:

```toml
# . or absolute path, please note that the directories following must be under root

root = "." 
tmp_dir = "tmp"

[build]
# Just plain old shell command. You could use `make` as well.
cmd = "go build -o ./myprog main.go"
# Binary file yields from `cmd`.
bin = "myprog"
# Customize binary, this is how you start to run your application
full_bin = "./myprog"

# This log file places in your tmp_dir, only build error logs will be put there
log = "build-error.log"

# Watch these filename extensions.
include_ext = ["go", "yaml", "html"]
# Ignore these filename extensions or directories.
exclude_dir = ["tmp"]
# It's not necessary to trigger build each time file changes if it's too frequent.
delay = 1000 # ms

[log]
# Show log time
time = true

[misc]
# Delete tmp directory on exit
clean_on_exit = true
```

Then under your project folder, you can just run `$ air`. When you change something like your .go and .html file, it will auto reload. One thing I don't like though is there is a glitch window, b/c air needs to shutdown the old process and start the new one, if you refresh page in this glitch window, your page will be broken. You will need to wait for 1 second to refresh again after the new process started, although it is a short time window, but it is still annoying when you experienced lots of this case. 



## Reference

* [Fiber Air Config File Sample](https://github.com/gofiber/recipes/blob/master/air/.air.linux.conf)
* [Air Config Example](https://github.com/cosmtrek/air/blob/master/air_example.toml)

