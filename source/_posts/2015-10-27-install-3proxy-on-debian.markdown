---
layout: post
title: "在Debina VPS上安装3proxy，支持HTTP、HTTPS、SOCKS代理"
date: 2015-10-27 11:26:27 +0800
comments: true
categories: ["VPS", "翻墙"]
---

[3proxy](https://3proxy.ru/)是俄罗斯人写的代理软件。  

首先在debian上安装，推荐[安装脚本](https://github.com/benjamin74/3proxy)：  

```
wget --no-check-certificate https://raw.github.com/benjamin74/3proxy/master/3proxyinstaller.sh
chmod +x 3proxyinstaller.sh
./3proxyinstaller.sh
  
```

然后编辑设置：  

```
vim /etc/3proxy/3proxy.cfg
```

修改登录帐号密码，将users行改为如下内容，后面即可使用用户名user1/密码passwd1，或用户名user2/密码passwd2登录：  

```
users user1:CL:passwd1
users user2:CL:passwd2
```

修改代理端口，在3128端口开启HTTP和HTTPS代理，1080端口开启SOCKS代理：  

```
proxy -a -p3128
socks -a -p1080
```

启动3proxy：  

```
/etc/3proxy/3proxy /etc/3proxy/3proxy.cfg &
```

另外安装脚本已经自带开机启动设置了。

更多设置参考[3proxy manual](https://3proxy.ru/doc/man3/3proxy.cfg.3.html)

Over