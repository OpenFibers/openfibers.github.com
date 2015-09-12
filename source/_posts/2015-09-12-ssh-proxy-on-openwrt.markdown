---
layout: post
title: "在OpenWRT上搭建ssh代理"
date: 2015-09-12 14:21:41 +0800
comments: true
categories: 
---

首先为何要用ssh代理，而不是ShadowSocks或VPN：GFW会针对ShadowSocks或各种VPN协议做解析，却不一定有勇气禁止全部ssh连接（国家曾经有次物理断开到国外的全部网络连接造成了巨大经济损失）。  

前段时间开发了[SSHMole](http://github.com/openfibers/sshmole)，用作OS X上的ssh代理客户端。然而MAC给iPhone开ssh代理，或在iPhone上直接开ssh代理还是很麻烦的。所以在家里路由器上开了个ssh -D。  
### 0. 首先要有一台VPS
没有的话到[vpsadd](http://vpsadd.com)买一台。

### 1. OpenWRT上安装ssh-client, openssh-keygen

登录到OpenWRT，移除ssh到dropbear的软连接
```
mv ssh dropbear-ssh
mv scp dropbear-scp
```

<!--more-->

安装：  
```bash
opkg install openssh-client openssh-keygen
```

在极路由上，openssh-keygen需要加入其他软件源（我是懒得装了，直接在OS X里生成了identity，scp到了OpenWRT和VPS上）；另外极路由默认已安装openssh-client了，直接将其软链到ssh和scp:  
```
ln -s /usr/bin/openssh-ssh /usr/bin/ssh
ln -s /usr/bin/openssh-scp /usr/bin/scp
```

### 2. 在OpenWRT生成rsa密钥对，复制到VPS

生成identity：  

```bash
ssh-keygen -b 1024 -t rsa
```

复制id_rsa.pub到VPS的~/.ssh/openwrt.pub：  

```bash
scp ~/.ssh/id_rsa.pub username@vps_host:~/.ssh/openwrt.pub
```

然后连接到vps后，将openwrt.pub加入到authorized_keys：  

```bash
cat ~/.ssh/openwrt.pub >> ~/.ssh/authorized_keys
```

回到openwrt，测试ssh是否能免密码登录：  
```bash
ssh user@vps_host -i ~/.ssh/id_rsa
```

### 3. 安装autossh

```bash
opkg install autossh
```

测试autossh是否能正常启动：  
```
autossh -M20000 -f -i ~/.ssh/id_rsa -Ng -D 7080 user@vps_host
ps|grep autossh
```

### 4. 设置autossh开机启动

在init.d下建立文件proxy_vps：  
```
touch /etc/init.d/proxy_vps
chmod +x /etc/init.d/proxy_vps
vim /etc/init.d/proxy_vps
```

在里面填入如下内容（在极路由上直接启动autossh会失败，先sleep一会就能启动了，不知为何）：  
```bash
#!/bin/sh /etc/rc.common
START=99
start()
{
	sleep 5
	autossh -M20000 -f -i ~/.ssh/id_rsa -Ng -D 7080 user@vps_host
}
stop()
{
	killall autossh
}
```

然后启用开机自动运行：  
```bash
/etc/init.d/proxy_vps enable
```

### 5. 增加pac文件

OpenWRT自带uHTTPd，直接将pac文件scp到/www下即可：  

```bash
scp blacklist.pac root@openwrt:/www
```