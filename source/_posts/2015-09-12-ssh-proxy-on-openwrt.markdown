---
layout: post
title: "在OpenWRT搭建ssh代理"
date: 2015-09-12 14:21:41 +0800
comments: true
categories: 
---

前段时间开发了SSHMole，用作OS X上的ssh代理客户端。然而iPhone上直接开ssh -D还是很麻烦的。所以准备在家里路由器上开个ssh -D。    
至于为何要用ssh代理，而不是ShadowSocks或VPN：GFW会针对ShadowSocks或各种VPN协议做解析，却不一定有勇气禁止全部ssh连接（国家曾经有次物理断开到国外的全部网络连接造成了巨大经济损失）。  

###1. 在OpenWRT生成rsa公钥，复制到VPS

登录到OpenWRT，然后生成identity。OpenWRT默认的ssh客户端是dropbear，直接使用dropbearkey生成key：  

```bash
dropbearkey -t rsa -f ~/.ssh/id_rsa
```

导出公钥到authorized_keys：  

```bash
dropbearkey -y -f ~/.ssh/id_rsa | grep "^ssh-rsa " >> authorized_keys
chmod 600 authorized_keys
```

复制authorized_keys到VPS：  

```bash
scp authorized_keys root@vps_host:vps_user_home/.ssh
```

测试ssh是否能免密码登录
```bash
ssh user@vps_host -i ~/.ssh/id_rsa
```