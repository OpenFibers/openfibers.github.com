---
layout: post
title: "Debug iOS https traffic"
date: 2016-03-18 14:39:33 +0800
comments: true
categories: ['iOS', 'debug', 'BurpSuite']
---

### Introduction
开发中我们经常会需要对HTTP/HTTPS请求进行抓包。  
抓包实际上是在中间机器开了一个代理服务，让需要抓包的请求经过代理，我们就可以看到这些请求了。本质上是中间人攻击。  
BurpSuite是一个常用的调试工具。  

### 下载BurpSuite
从[BurpSuite](http://portswigger.net/burp/download.html)官网下载jar包，右键点击，运行：  

{% img left /images/blog/ios_burp/BurpSuite_1.jpg 947 %}  

<!--more-->

### Burp设置

先从菜单Burp->Remember settings中检查是否All options都记录设置了，以便下次打开不用重新配置：  

{% img left /images/blog/ios_burp/BurpSuite_2.jpg 418 %}  

在选项卡的Proxy->Options中，选择代理规则，点击Edit：  

{% img left /images/blog/ios_burp/BurpSuite_3.jpg 693 %}  

在弹出的对话框中选择All interfaces，再点击OK，来监听所有的网卡：  

{% img left /images/blog/ios_burp/BurpSuite_4.jpg 707 %}  

至此已经可以通过代理来监听手机的HTTP请求了。现在我们再制作CA让手机信任，来解密被加密的HTTPS请求。  

回到选项卡的Proxy->Options中，重新生成证书，以防被拥有相同的证书的人中间人攻击。生成后需要重启Burp：  

{% img left /images/blog/ios_burp/BurpSuite_5.jpg 633 %}  

重启Burp后，回到选项卡的Proxy->Options中，导出证书为Der格式：  

{% img left /images/blog/ios_burp/BurpSuite_6.jpg 946 %}  

然后将Der证书通过HTTP服务器或邮件发给手机，在手机上安装证书：  

{% img left /images/blog/ios_burp/InstallCert_1.jpg 250 %}
{% img left /images/blog/ios_burp/InstallCert_2.jpg 250 %}
{% img left /images/blog/ios_burp/InstallCert_3.jpg 250 %}

第一次用，先关闭排除规则，抓取全部的包。在Proxy->Intercept选项卡中，点击按钮，使其显示‘Intercept is off’:  

{% img left /images/blog/ios_burp/BurpSuite_7.jpg 571 %}  

### 设置手机/模拟器代理

先看下Mac的网卡地址：  

{% img left /images/blog/ios_burp/ProxySettings_1.jpg 664 %}  

然后在手机的wifi详情中设置手动代理：  

{% img left /images/blog/ios_burp/ProxySettings_2.jpg 375 %}  

如果是模拟器，在网卡的高级设置中，设置HTTP和HTTPS代理为127.0.0.1:8080：  

{% img left /images/blog/ios_burp/ProxySettings_3.jpg 662 %}  

在手机或模拟器中发送请求，然后在Burp选项卡的Proxy->HTTP history中可以查看到结果：  

{% img left /images/blog/ios_burp/BurpSuite_8.jpg 857 %}  

### 测试完成后删除手机的Der

不删除一旦私钥泄露会被中间人攻击，保险起见，调试完就赶紧从手机删掉证书。  

在系统设置->通用->描述文件中找到刚才安装的证书，然后删除：  

{% img left /images/blog/ios_burp/RemoveCert_1.jpg 375 %}
{% img left /images/blog/ios_burp/RemoveCert_2.jpg 375 %}  


Over
