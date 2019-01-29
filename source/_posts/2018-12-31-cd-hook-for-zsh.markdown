---
layout: post
title: "zsh中对cd命令的hook"
date: 2018-12-31 12:45:47 +0800
comments: true
categories: [shell, macOS]
---

zsh中预留了一个`chpwd()`用于hook各种cd命令，包括`cd` `pushd` `popd`等。

首先打开`~/.zshrc`，增加`chpwd`函数。作用是在ccproj或其子目录，使用全局代理，否则关闭代理。

```
chpwd()
	case $PWD in
		(*/ccproj)
			echo using proxy
			allproxy
			;;
		(*/ccproj/*)
			echo using proxy
			allproxy
			;;
		(*)
			noproxy
			;;
	esac
```

其中`allproxy`和`noproxy`的定义如下:  

<!--more-->

```
alias allproxy='export ALL_PROXY=socks5://127.0.0.1:9050'
alias noproxy='export ALL_PROXY='
```

经过设置，每次切换路径就会自动切换代理。但是新开的terminal窗口是不会被hook的。我们手动在`~/.zshrc`中增加新窗口打开后原地执行一次`cd .`命令，修改后的`~/.zshrc`如下:

```
alias allproxy='export ALL_PROXY=socks5://127.0.0.1:9050'
alias noproxy='export ALL_PROXY='

chpwd()
	case $PWD in
		(*/ccproj)
			echo using proxy
			allproxy
			;;
		(*/ccproj/*)
			echo using proxy
			allproxy
			;;
		(*)
			noproxy
			;;
	esac

cd .
```

以后就可以随意切路径不会忘了改代理了。  
Over