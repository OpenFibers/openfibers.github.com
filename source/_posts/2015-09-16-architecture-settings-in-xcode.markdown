---
layout: post
title: "Architectures 与 Valid Architectures 在 Xcode 中的设置"
date: 2015-09-16 10:39:18 +0800
comments: true
categories: 
---

### 关于这几个设置项，先看官方说明：  

1. Architecture:  
Space-separated list of identifiers. Specifies the architectures (ABIs, processor models) to which the binary is targeted. When this build setting specifies more than one architecture, the generated binary may contain object code for each of the specified architectures.


2. Vaild Architecture:  
Space-separated list of identifiers. Specifies the architectures for which the binary may be built. During the build, this list is intersected with the value of ARCHS build setting; the resulting list specifies the architectures the binary can run on. If the resulting architecture list is empty, the target generates no binary.

3. Build Active Architecture Only:  
Boolean value. Specifies whether the product includes only object code for the native architecture.

<!--more-->

### 关于Architecture 和 Valid Architecture
简单来说，Architecture 和 Valid Architecture 的交集，是最终编译的target。  
举例：Architecture 设置为armv7 arm64，Valid Architecture 设置为 armv7，工程中代码则只会编译到 armv7，然后 link。Architecture 设置为 armv7，Valid Architecture 设置为 armv7 arm64，工程中代码也只会编译到 armv7，然后 link。  

另可以使用 $(ARCHS_STANDARD) 指定当前流行的 target。Xcode6.4，Xcode7中 iOS 对应的是 armv7 arm64，OSX 对应的是 x86_64。  

一般来说，iOS app、OSX app、OSX 开源库或 OSX SDK 的 target 将 Architecture 和 Vaild Architecture 均设置为$(ARCHS_STANDARD)即可。  
iOS 开源库或 SDK 则需设置为$(ARCHS_STANDARD) 和 i386，以兼容 iOS 模拟器。  

### 关于Build Active Architecture Only:  
如果设置为 YES，连接设备调试时（包括 iOS 设备和 Mac 设备），则只编译到此设备的指令集。如果Architecture 和 Vaild Architecture 的交集中不含此设备的指令集时，Xcode6.4实际测试结果是编译失败，找不到对应设备指令集的.o 文件，产生 link error。  
设置为 NO 或未连接设备调试时，则编译到 Architecture 和 Vaild Architecture 的交集。  

Over