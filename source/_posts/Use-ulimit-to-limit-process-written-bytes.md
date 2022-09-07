---
title: Use ulimit to limit process written bytes
typora-root-url: ../../source
date: 2022-09-07 14:31:37
tags: [ulimit,linux,ubuntu]
---

## Problem

You want to limit a process to only use limited resources in your server. 



## Solution

The `ulimit` can help with that. You can check this with the `ulimit --help` command in linux:

```bash
$ ulimit --help
    Options:
      -S	use the `soft' resource limit
      -H	use the `hard' resource limit
      -a	all current limits are reported
      -b	the socket buffer size
      -c	the maximum size of core files created
      -d	the maximum size of a process's data segment
      -e	the maximum scheduling priority (`nice')
      -f	the maximum size of files written by the shell and its children
      -i	the maximum number of pending signals
      -k	the maximum number of kqueues allocated for this process
      -l	the maximum size a process may lock into memory
      -m	the maximum resident set size
      -n	the maximum number of open file descriptors
      -p	the pipe buffer size
      -q	the maximum number of bytes in POSIX message queues
      -r	the maximum real-time scheduling priority
      -s	the maximum stack size
      -t	the maximum amount of cpu time in seconds
      -u	the maximum number of user processes
      -v	the size of virtual memory
      -x	the maximum number of file locks
      -P	the maximum number of pseudoterminals
      -R	the maximum time a real-time process can run before blocking
      -T	the maximum number of threads
```

Some options won't work in certain platform. For example, if you use `ulimit -T` in linux, it will show "invalid option" error. If you further wonder how do you then limit "number of threads" in linux, you can use the `-u` option, which limit the maximum number of user processes, because in linux a process and a thread are essentially the same, both called "task" inside the kernel.



You can set the soft limit and hard limit for a certain resource with the `-S` or `-H` command. If you want to set both, could just use `-SH`, for example, if you want to set the soft and hard limit of "maximum open FDs" to be 100, you could use `ulimit -SHn 100`. Internally, it is using this struct:

```c
#include <sys/resource.h>

struct rlimit {
  rlim_t rlim_cur;	// soft limit
  rlim_t rlim_max;	// hard limit
}
```



## Example

### 1) Set open files number

To get the current ulimit value, you can use `ulimit -a`. To see specific values, for example, the "maximum open FD", you can use `ulimit -Sn` for the soft limit, and `ulimit -Hn` for the hard limit.

Anothe example to get the limit with clang:

```c
#include <sys/resource.h>
#include <stdio.h>

int main() {
    struct rlimit rlim;
    if (getrlimit(RLIMIT_NOFILE, &rlim) == -1) {
        return -1;
    }
    printf("soft: %lld \n", (long long) rlim.rlim_cur);
    printf("hard: %lld \n", (long long) rlim.rlim_max);
    return 0;
}
```

Compile and run:

```bash
$ gcc main.c -o main
$ ./main
soft: 1024
hard: 1048576
```

You can set the limit with an extra paramter, like `ulimit -Sn 2000`, now if you use `ulimit -Sn`, it will show 2000.



### 2) Set written file size

To see the current written file size, use `ulimit -Sf`, it shows "unlimited". If you use `ulimit -a`, it shows things like this:

```bash
file size                   (blocks, -f) unlimited
```

So if we set the number for this, e.g. 10, the limited written file size will be **10 blocks**. Now the question is, what is the size of "a block"? To test this out, let's have a python script first:

```python
#!/usr/bin/env python3

with open("/tmp/asdf.txt", "w") as f:
    f.write("0" * 1024)
```

Here we will write 1024 bytes to a file. Now let's limit the "file size" to 1 block and run this file:

```bash
$ ulimit -Sf 1
$ python3 writefile.py
```

No error. Now let's change 1024 to 1025 in the above file, and run `python3 writefile.py` again:

```bash
$ python3 writefile.py
OSError: [Errno 27] File too large

During handling of the above exception, another exception occurred:

Traceback (most recent call last):
  File "/home/johndoe/project/ulimit/writefile.py", line 3, in <module>
    with open("/tmp/asdf.txt", "w") as f:
OSError: [Errno 27] File too large
```

Ooops, now we have an error, apparently it exceeded the "max file size" limit. So now we know the "block" size is 1024 bytes.

> The block size is operation system dependant. The above code is tested in Ubuntu 22.04 x86_64. I also tested the same code in macOS Monterey (version 12.5), in which the block size is actually 512 bytes.

To set the limit back to unlimited, use `ulimit -Sf unlimited`.



## An unsolved problem

I tried the `Getrlimit` with Golang (version 1.19.1), in the same computer:

```golang
package main

import (
    "fmt"
    "syscall"
)

func main() {
    lim := syscall.Rlimit{}

    err := syscall.Getrlimit(syscall.RLIMIT_NOFILE, &lim)
    if err != nil {
        panic(err)
    }
    fmt.Println("soft:", lim.Cur)
    fmt.Println("hard:", lim.Max)
}
```

Run:

```bash
$ go run main.go
soft: 1048576
hard: 1048576
```

While the C code above gave:

```c
soft: 1024
hard: 1048576
```

I don't know why this happened 😟, here is the [source code >>](https://cs.opensource.google/go/go/+/master:src/syscall/syscall_linux_386.go;l=83?q=Getrlimit&sq=&ss=go%2Fgo)



## Reference

* https://cloud.tencent.com/developer/article/1661160
* https://github.com/golang/go/issues/5949
* https://pkg.go.dev/syscall
* https://linuxhint.com/linux_ulimit_command/
* https://phoenixnap.com/kb/ulimit-linux-command

