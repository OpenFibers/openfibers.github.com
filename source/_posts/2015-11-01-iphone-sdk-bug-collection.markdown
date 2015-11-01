---
layout: post
title: "iPhone SDK Bug Collection"
date: 2015-11-01 21:01:42 +0800
comments: true
categories: ['iPhone', 'iOS']
---

1. convertRect:fromView 返回乘以screen scale以后的结果：  
在iOS8上，view如果未添加到任何window，调用此方法可能会出现此后果。  