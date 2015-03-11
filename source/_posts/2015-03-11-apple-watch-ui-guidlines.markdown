---
layout: post
title: "Apple Watch人机交互指南"
date: 2015-03-11 13:49:29 +0800
comments: true
categories: 
---

[官方文档传送门](https://developer.apple.com/library/prerelease/ios/documentation/UserExperience/Conceptual/WatchHumanInterfaceGuidelines/index.html)

## 为Apple Watch设计软件的注意事项：
1. 私人的。带有心率等其它传感器，由于是苹果的首个穿戴式设备，在设计应用时要时刻想着apple watch是和使用者紧密相连的。  
2. Apple watch是软件与硬件的结合。应善用旋钮、压力感应触摸屏。  
3. Apple watch上的软件定义为轻量级的，要求能快速启动，交互简洁、UI简单。  

## Apple Watch App详解

### UI类型
App有两种基本样式，基于继承的、基于平行页面的。  

1. Hierarchical：更像iOS上的导航模式。有推入推出。适合处理复杂的业务逻辑。  
2. Page-based：几个页面横滑切换，而且苹果希望尽可能少的并列页面。适合处理简单的业务逻辑。  

这两种基本架构不能同时存在，只能二选其一。  

### 用户交互

1. Action-based：单击操作。在各种各样的控件上点的事件都是。  
2. Gestures：竖滑、横滑、左边缘右滑后退、点击。  
3. Force Touch：大力按下时系统会弹出context menu，作用类似pc上的右键菜单。App内当使用此menu显示上下文可以操作的actions。  
4. The Digital Crown：第三方应用仅可以使用表冠卷动来支持滚动长页面。  