---
layout: post
title: "macOS 卸载 Symantec 和 McAfee"
date: 2019-07-30 11:02:04 +0800
comments: true
categories: 
---

Activity Monitor 查一下相关几个进程的 pid:

。。进程名我忘了记录了，大约就是 Mc 和 Sym 开头的几个进程

lsof 看一下 pid 相关的 file handle:  

```
lsof -p pid_of_McAfee -Fn | grep Mc
lsof -p pid_of_Symantec -Fn | grep Sym
```

记录 McAfee 和 Symantec 的 executable file 位置：

```
/Applications/Symantec Solutions/*
/Applications/McAfee Endpoint.app
/Library/Application Support/Symantec
/Library/Application Support/McAfee
/usr/local/McAfee
/private/var/McAfee
```

关机，长按 command + S, 进入 macOS single user mode, 检查磁盘并挂载磁盘：  

```
fsck -fy
mount -uw /
```

删除 McAfee 和 Symantec 相关文件：  

```
rm -rf /Applications/Symantec Solutions/*
rm -rf /Applications/McAfee Endpoint.app
rm -rf /Library/Application Support/Symantec
rm -rf /Library/Application Support/McAfee
rm -rf /usr/local/McAfee
rm -rf /private/var/McAfee
```

重启：

```
reboot
```

开机 Activity Monitor 里看看还有没有相关进程，如果有，从文章最开始再来一遍。。。

Over