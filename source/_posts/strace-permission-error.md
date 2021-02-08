---
title: 'strace permission error'
date: 2018-10-04 02:01:58
tags: [linux]
published: true
hideInList: false
feature: /post-images/strace-permission-error.png
---
When run strace command in Ubuntu, I got an error:
<!-- more -->


> strace: Could not attach to process. If your uid matches the uid of the target process, check the setting of /proc/sys/kernel/yama/ptrace_scope, or try again as the root user. For more details, see /etc/sysctl.d/10-ptrace.conf: Operation not permitted
> strace: attach: ptrace(PTRACE_SEIZE, 7604): Operation not permitted

Solution 1: Run as root

Solution 2: `sudo sysctl -w kernel.yama.ptrace_scope=0`