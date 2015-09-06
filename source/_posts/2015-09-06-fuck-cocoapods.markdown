---
layout: post
title: "难用的CocoaPods"
date: 2015-09-06 10:50:14 +0800
comments: true
categories: ['iOS','CocoaPods']
---

前几天从网易云音乐离职，加入了 Rush。然后新 app 用了CocoaPods。  
老实说，我对这个软件一直挺抵触的。做大 app 时，这个东西并没什么卵用，维护 pod 的时间很可能比手动维护几个 lib 还要时间长。这个软件的最佳使用场景，按我的理解，做外包频繁创建新工程时，可能会比较顺手。  
然后在同事机器上，pod 工作的很正常，到了我的机器上，就 link error，原因是他的 pod 自动生成的.a 和我的 pod 自动生成的 .a 带的前缀不一样。  

<!--more-->

解决过程也很梦幻，看了同事的 CocoaPods版本是 0.37.1，我的是 0.38.0，于是卸载了 0.38.0，装了 0.37.1，再更新 pod，就康复了：  


```bash
#删除当前版本pod，安装指定版本的pod
sudo gem uninstall cocoapods
sudo gem install cocoapods -v 0.37.1

#清空pod缓存后更新由pod管理的libs
rm -rf "${HOME}/Library/Caches/CocoaPods"
rm -rf "`pwd`/Pods/"
pod update
```

参考[mbinna/podforceupdate.sh](https://gist.github.com/mbinna/4202236)。  