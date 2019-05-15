---
layout: post
title: "JDK 和 Maven macOS 的安装与配置"
date: 2019-05-15 20:53:13 +0800
comments: true
categories: 
---

# JDK

### 下载安装

到 [https://www.oracle.com/technetwork/java/javase/downloads/index.html](https://www.oracle.com/technetwork/java/javase/downloads/index.html) 下载想用的 jdk 版本，需要登录。

下载好以后双击 dmg 安装。

命令行输入 `java -version` 查看版本。

### 配置 JAVA_HOME 环境变量

安装包没自动配置环境变量，可能是考虑宿主机多个不同 jdk 版本，没替用户做决定，需要手动配置一下。  

<!--more-->

首先看一下实际的 java home: 

```bash
$ /usr/libexec/java_home -V
Matching Java Virtual Machines (1):
    1.8.0_211, x86_64:  "Java SE 8" /Library/Java/JavaVirtualMachines/jdk1.8.0_211.jdk/Contents/Home

/Library/Java/JavaVirtualMachines/jdk1.8.0_211.jdk/Contents/Home
```

拿到路径 `/Library/Java/JavaVirtualMachines/jdk1.8.0_211.jdk/Contents/Home` 后，打开 shell profile 修改配置：  

```
$ vim ~/.zshrc
```

增加配置：  

```bash
export JAVA_HOME=`/usr/libexec/java_home`
export PATH=$JAVA_HOME/bin:$PATH
```

### 验证

```
echo $JAVA_HOME
```

# Maven

### 下载

[https://maven.apache.org/index.html](https://maven.apache.org/index.html)

### 安装

```
unzip apache-maven-3.6.1-bin.zip
or
tar xzvf apache-maven-3.6.1-bin.tar.gz
sudo cp -r apache-maven-3.6.1 /opt
vim ~/.zshrc
```

```
export PATH=/opt/apache-maven-3.6.1/bin:$PATH
```

### 验证

```
mvn -v
```


