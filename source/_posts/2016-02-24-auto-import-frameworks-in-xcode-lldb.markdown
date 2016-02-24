---
layout: post
title: "Xcode/lldb自动导入UIKit"
date: 2016-02-24 10:22:27 +0800
comments: true
categories: ['Xcode', 'lldb', 'iOS', 'OSX']
---

## lldb导入UIKit

使用lldb调试app时，lldb经常提示view.bounds无法打印：  

{% img left /images/blog/lldb_auto_import/lldb_auto_import1.png 428 58 %}  

实际上是lldb启动时没有默认导入UIKit，所以UIKit中的方法无法识别。在lldb中使用@import手动导入

```
expr @import UIKit
```

即可解决：  

{% img left /images/blog/lldb_auto_import/lldb_auto_import2.png 396 57 %}  

在开发Mac App时也是一样的，比如不导入AppKit则无法打印frame属性，手动导入AppKit即可：  

```
expr @import AppKit
```

{% img left /images/blog/lldb_auto_import/lldb_auto_import3.png 297 113 %}  

## 工程自动导入

但是大部分人都换有懒癌，对于每次调试都要手打一行命令是难以接受的。我们可以在main上加个断点：  

{% img left /images/blog/lldb_auto_import/lldb_auto_import4.png 799 113 %}  

在main中断时执行`expr @import UIKit`，并自动continue:  

{% img left /images/blog/lldb_auto_import/lldb_auto_import5.png 647 370 %}  

## 全局自动导入

对于懒癌晚期的患者，每个工程单独配置还是挺麻烦的，可以在lldbinit中设置全局自动导入UIKit。在命令行中执行：  

```bash
echo display @import UIKit >> ~/.lldbinit
echo display @import AppKit >> ~/.lldbinit
```

再重新启动调试即可（Xcode7测试时无需重启Xcode）。  

Over
