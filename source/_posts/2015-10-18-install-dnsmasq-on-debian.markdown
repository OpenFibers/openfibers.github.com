---
layout: post
title: "在Debian VPS上安装dnsmasq解决DNS污染"
date: 2015-10-18 23:34:24 +0800
comments: true
categories: ["VPS", "翻墙"]
---

warning: 我的有台DNS因为装了dnsmasq没做加密，被服务商提示有被DDoS攻击风险，强制断线了。  

### 1. 安装

首先连上VPS，安装dnsmasq：  
```bash
sudo apt-get install dnsmasq
```

我的VPS连接debian的APT服务器IPv6地址连不上，如果同样卡在连接 http://http.debian.net 不动的话，建议关掉IPv6再重试安装。  

### 2. 配置

打开配置文件：  
```bash
vim /etc/dnsmasq.conf
```

填入以下设置，将google的服务器作为上级DNS：  

```bash
server 8.8.8.8
server 8.8.4.4
```

较新版本dnsmasq只破DNS污染，无需特殊服务的话，没必要更改其他设置，直接重启即可：  
```bash
service dnsmasq restart
```

<!--more-->

### 3. VPS上测试服务是否已开启

查看dnsmasq是否已使用了53端口：  
```bash
netstat -tunlp|grep 53
```

然后就可以在客户机配置VPS地址作为DNS了。  

### 4. OSX上设置VPS作为DNS服务器

1. 打开System Preference，在Network->网卡的Advanced->DNS，将VPS地址填入即可。  
2. 随后在Terminal中可测试是否设置成功：  
```bash
scutil --dns | grep 'nameserver\[[0-9]*\]'
```
或者
```bash
cat /etc/resolv.conf
```

3. 打开Network Utility，在Lookup选项卡中可测试域名查询到的IP地址。  

Over