---
layout: post
title: "iPhone SDK Bug Collection"
date: 2015-11-01 21:01:42 +0800
comments: true
categories: ['iPhone', 'iOS', 'UIKit']
---

1. convertRect:fromView 返回乘以screen scale以后的结果：  
在iOS8上，view如果未添加到任何window，调用此方法可能会出现此后果。  

2. 定高UITextView输入时，文字超过text view高度，输入光标被遮挡或截断，显示不全，继续输入时正在输入的内容也被遮挡:  
参考[iOS8的UITextView输入光标显示不全的hack](http://openfibers.github.io/blog/2015/11/30/uitextview-auto-scroll-in-ios8/)。  

3. window.rootViewController设置后，老的rootViewController的view仍然贴在window上：  
参考[[UIWindow setRootViewController:]后view无限叠加的问题修复](http://openfibers.github.io/blog/2015/12/15/window-setrootviewcontroller-view-not-removed-hack/)。  