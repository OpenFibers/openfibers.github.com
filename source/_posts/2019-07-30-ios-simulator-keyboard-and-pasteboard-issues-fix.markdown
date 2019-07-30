---
layout: post
title: "iOS 模拟器键盘无响应和剪贴板不响应问题修复(Xcode 10.3)"
date: 2019-07-30 15:28:07 +0800
comments: true
categories: 
---

## 键盘无响应

iOS Simulator main menu - Hardware - Keyboard  
首先取消 Use the Same Keyboard Language as macOS  
点击 Connect Hardware Keyboard  

## 剪贴板无响应

iOS Simulator 使用的剪贴板服务挂了，还是 main menu   
Edit - 取消 Automatically Sync Pasteboard  
再 Edit - 选中 Automatically Sync Pasteboard  

Over
