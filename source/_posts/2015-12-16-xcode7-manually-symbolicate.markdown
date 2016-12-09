---
layout: post
title: "Xcode 中手动 Symbolicate Crash Log"
date: 2015-12-16 10:45:23 +0800
comments: true
categories: ['iOS', 'macOS', 'Xcode']
---

从 Xcode7 开始 Organizer 集成了查看崩溃日志的功能，若 Xcode 可以定位到 crash 的 build，则可以直接在 Organizer 中查看崩溃是在具体哪一行。  
但是不能自动定位到 build 的情况也非常常见，此时地址不能自动转换成符号：  

{% img left /images/blog/xcode7_manually_symbolicate1/xcode7_manually_symbolicate1.JPG %}  

此时即需要手动转换。  

<!--more-->

#### 1. 将.app文件夹、dSYM文件夹、crash文件放在同一个路径下

{% img left /images/blog/xcode7_manually_symbolicate1/xcode7_manually_symbolicate2.JPG %}  

其中.app文件夹也可以通过解压ipa文件提取到。  

#### 2. 配置Xcode环境变量

```bash
export DEVELOPER_DIR="/Applications/Xcode.app/Contents/Developer"
```

#### 3. 为了方便，把Xcode提供的symbolicatecrash工具加一个alias

Xcode8 :  
```
alias symbolicatecrash='/Applications/Xcode.app/Contents/SharedFrameworks/DVTFoundation.framework/Versions/A/Resources/symbolicatecrash'
```

Xcode7 / Xcode6 :  
```
alias symbolicatecrash='/Applications/Xcode.app/Contents/SharedFrameworks/DTDeviceKitBase.framework/Versions/Current/Resources/symbolicatecrash'
```

Xcode5 :  
```
alias symbolicatecrash ='/Applications/Xcode.app/Contents/Developer/Platforms/iPhoneOS.platform/Developer/Library/PrivateFrameworks/DTDeviceKitBase.framework/Versions/Current/Resources/symbolicatecrash'
```

#### 4. 切换到刚刚放置.app文件夹、dSYM、crash log的路径

```
cd CRASH_FILES_PATH
symbolicatecrash -v crash_to_convert.crash > converted.crash
```

如果一切正常，converted.crash即为转换后的可读的crash log。  

#### 5. 如果没有成功，则检查.app、dSYM和crash log的build UUID是否相同

首先查看可执行文件和dSYM的build UUID：  

```bash
➜ dwarfdump --uuid Crash.app/Crash
UUID: 3ADB495D-8680-3403-A117-360D7658DA30 (armv7) Crash.app/Crash
UUID: 5105C8AA-D5A1-31B7-A0B5-309EB899F43F (arm64) Crash.app/Crash
➜ dwarfdump --uuid Crash.app.dSYM/Contents/Resources/DWARF/Crash 
UUID: 3ADB495D-8680-3403-A117-360D7658DA30 (armv7) Crash.app.dSYM/Contents/Resources/DWARF/Crash
UUID: 5105C8AA-D5A1-31B7-A0B5-309EB899F43F (arm64) Crash.app.dSYM/Contents/Resources/DWARF/Crash
```

将 UUID 去掉减号'-'在 crash log 中搜索，如果 crash log 中 binary image 的 UUID 和可执行文件、dSYM的build UUID一致，则可以确定此 crash log 崩溃自此 build。否则 crash log 和此 build 不匹配。  

{% img left /images/blog/xcode7_manually_symbolicate1/xcode7_manually_symbolicate3.JPG %}  

Over  