---
layout: post
title: "CocoaPods 分支依赖时的 bug"
date: 2019-07-22 15:22:31 +0800
comments: true
categories: 
---

主工程源码依赖 SDK develop 分支，`pod install` 和 `pod install --fast-mode` ，拉下来的 SDK 均不是 develop 分支最新提交，而是上次执行 `pod install` 时 develop 分支的提交。  

清空 `./Pods` 和 `~/Library/Caches/CocoaPods/*` 均无效（缓存不在这里）。  

由于下载时间太长，不想清掉全量本地库，于是想了一个变通的方法。  

SDK 里 `gco -b feature/merge_main_proj`，主工程依赖 SDK `feature/merge_main_proj` 分支，重新 `pod install --fast-mode` 就好了，因为 pods 里没有对 `feature/merge_main_proj` 的缓存，此时肯定会好。  

然后主工程切回对 SDK `develop` 分支的依赖，重新 `pod install --fast-mode` 也好了。估计是对一个 pod 的多个分支只有一份缓存（pods 版本 1.5.0）。  

Over