---
layout: post
title: "在Windows上安装Ruby"
date: 2015-03-04 22:18:27 +0800
comments: true
categories: ["Ruby"]
---

Mac上使用RVM[https://rvm.io]安装与管理各种版本的Ruby是非常简单易行的。今天心血来潮想在Windows上写Octopress，安装了Ruby，似乎已经没有13年的时候那么难装了。

1. 下载安装包
访问[RubyInstaller下载页](http://rubyinstaller.org/downloads/)，找到适合版本的Ruby Installer和Development Kit。

2. 安装RubyInstaller：基本上只要双击安装包一直点下一步就可以了。

3. 安装Development Kit：双击解压下载到的7z自解压文件，将其解压到自己喜欢的路径下。

4. 按照[Development Kit Wiki](https://github.com/oneclick/rubyinstaller/wiki/Development-Kit)的指引步骤完成Development Kit安装。  
一般首次安装只要执行以下命令就可以了：

```batch
$ ruby dk.rb init
$ ruby dk.rb install
```

5. 检查是否安装成功

```batch
$ gem install json --platform=ruby
# 成功则最后一行显示 1 gem installed
$ ruby -rubygems -e "require 'json'; puts JSON.load('[42]').inspect"
# 安装成功则显示 [42]
```

Over