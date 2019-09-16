---
layout: post
title: "修复 _adjustContentOffsetIfNecessary 导致 scroll view 闪动问题"
date: 2019-09-16 19:50:22 +0800
comments: true
categories: 
---

### 方法一

继承 scroll view，override _adjustContentOffsetIfNecessary 到空方法。

### 方法二

swizz scroll view _adjustContentOffsetIfNecessary 到其 category 的一个空方法中。

### 方法三

对应 UIViewController 的 automaticallyAdjustsScrollViewInsets 设置为 NO。