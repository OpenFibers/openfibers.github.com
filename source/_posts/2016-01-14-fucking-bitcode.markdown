---
layout: post
title: "日了狗的BitCode"
date: 2016-01-14 18:06:54 +0800
comments: true
categories: ['iOS', 'Xcode', 'BitCode']
---

1. framework/static lib生成困难,需要手动合并fat executable file
2. 必须用Archive不能直接Build，否则iTunes Connect报错还不告诉你原因
3. 包的体积不降反增