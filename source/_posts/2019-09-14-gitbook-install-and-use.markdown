---
layout: post
title: "Gitbook install and use"
date: 2019-09-14 13:01:59 +0800
comments: true
categories: 
---

### 首先安装 Node.js

[https://nodejs.org/en/download/](https://nodejs.org/en/download/)

### 然后安装 Gitbook

```
sudo npm install -g gitbook-cli
```

### 再安装 Calibre

Gitbook 的电子书格式转换需要调用 Calibre 的命令行工具。  

[https://calibre-ebook.com](https://calibre-ebook.com)

然后：  

```
ln -s /Applications/calibre.app/Contents/MacOS/ebook-convert ~/bin
```

### 最后切换到书的 git 目录下

```
gitbook mobi ./ ./book-name.mobi
```

Over