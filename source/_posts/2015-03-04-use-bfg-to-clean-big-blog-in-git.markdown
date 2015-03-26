---
layout: post
title: "使用BFG Repo-Cleaner清除git的历史错误提交"
date: 2015-03-04 14:50:59 +0800
comments: true
categories: ["git"]
---

Git中有时会不小心提交大文件或密码到repo中，然而使用git-filter-branch清理以往的全部提交是非常复杂的。  
今天介绍一个好用的工具[BFG Repo-Cleaner](https://rtyley.github.io/bfg-repo-cleaner/)，可以方便清理错误的二进制文件或密码文件提交。  

以下命令中所有的 bfg 是 java -jar bfg.jar 的alias。

### 1. 克隆一份repo到本地

```bash
git clone --mirror git@github.com:OpenFibers/openfibers.github.com.git
```

### 2. 执行bfg命令移除目标文件： 

从历史纪录中删除所有文件名是 'id_rsa' 或 'id_dsa' 的文件：

```bash
$ bfg --delete-files id_{dsa,rsa}  openfibers.github.com.git
```

从历史纪录中删除所有大于1M的二进制文件 :

```bash
$ bfg --strip-blobs-bigger-than 1M  openfibers.github.com.git
```

<!--more-->

Replace all passwords listed in a file (prefix lines 'regex:' or 'glob:' if required) with ***REMOVED*** wherever they occur in your repository :

```bash
$ bfg --replace-text passwords.txt  openfibers.github.com.git
```

Remove all folders or files named '.git' - a reserved filename in Git. These often become a problem when migrating to Git from other source-control systems like Mercurial :

```bash
$ bfg --delete-folders .git --delete-files .git  --no-blob-protection  openfibers.github.com.git
```

### 3. Reflog & Push Back to Remote Repo

```
$ cd openfibers.github.com.git
$ git reflog expire --expire=now --all && git gc --prune=now --aggressive
$ git push
```

[详细介绍及下载地址传送门](https://rtyley.github.io/bfg-repo-cleaner/)，需要安装JDK。

Over