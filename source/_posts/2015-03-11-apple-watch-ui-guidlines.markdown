---
layout: post
title: "Apple Watch人机交互指南"
date: 2015-03-11 13:49:29 +0800
comments: true
categories: 
---

[官方文档传送门](https://developer.apple.com/library/prerelease/ios/documentation/UserExperience/Conceptual/WatchHumanInterfaceGuidelines/index.html)

# 为Apple Watch设计软件的注意事项：

## Apple从未有过的穿戴式设备
1. Personal: 一方面更隐私，另一方面更个人化。带有心率等其它传感器，可以得到比以往设备更多的个人信息。由于是苹果的首个穿戴式设备，从未有过一款设备像apple watch一样如此与使用者紧密相连，这一点在设计应用时要多加注意。  
2. Holistic: Apple watch是软件与硬件的结合。新的交互方式包括Digtal Crown(表冠)、Taptic Engine(触觉反馈+细微声音反馈)和Force Touch(压力感应触摸屏)，[官方相关介绍传送门](https://www.apple.com/cn/watch/technology/)。设计App时应当把以上交互体验一并考虑在内。    
3. Lightweight: Apple watch上的软件定义为轻量级的，要求能快速启动，交互简洁、UI简单。  

## UI架构与交互方式

### UI类型
App有两种基本样式，基于继承的、基于平行页面的。  

1. Hierarchical：更像iOS上的导航模式。有推入推出。适合处理复杂的业务逻辑。  

{% img left /images/blog/apple_watch_guidelines/1.hierarchical_interface_2x.png 513 267 %}  

2. Page-based：几个页面横滑切换，而且苹果希望尽可能少的并列页面。适合处理简单的业务逻辑。  

{% img left /images/blog/apple_watch_guidelines/2.paged_interface_2x.png 679 267 %}  

这两种基本架构不允许同时存在，只能二选其一。  

### 用户交互

1. Action-based：单击操作。在各种各样的控件上点的事件都是。  
2. Gestures：竖滑、横滑、左边缘右滑后退、点击。  
3. Force Touch：大力按下时系统会弹出context menu，作用类似pc上的右键菜单。App内当使用此menu显示上下文可以操作的actions。  
4. The Digital Crown：第三方应用仅可以使用表冠卷动来支持滚动长页面。  

<!--more-->

## Glances

每个App可以创建自己的Glance，Glance应当是自己应用中最关键内容的展示。  
用户从安装的App中选出一些喜欢应用的Glance，这就组成了Glances。  
有点像iOS的Extension。  

{% img left /images/blog/apple_watch_guidelines/3.glances_2x.png 486 267 %}  

Glances的特点:  
1. Template-based: 使用Xcode选取合适的模板，然后根据模板设计合适的内容。  
2. Not scrollable: 不可滚动。只能显示一屏。  
3. Associated with a single action: 点击Glance的任意位置都是相同的action，此时跳入app后，然后进入恰当的页面。  
4. Optional: 不是所有的app都需要做这个。可以不做。  

## Notifications

Notifications是在apple watch上用于处理远程通知或本地通知的方便轻量的交互。  
无论是远程通知，还是本地通知，都分为两个阶段。第一个阶段：Short Look, 在通知到达的第一时间展示给用户，应谨慎选择内容，给用户提供最少量的信息，以保护用户隐私。如果用户低下手腕，Short Look会自动隐藏。如果用户点击Short Look或者抬起手腕，Look Look将会显示出来。Long Look应当显示通知的详细信息，而且Long Look只能被用户关闭。  

### Short Look Notifications

Short Look用于告知用户收取了通知，并展示简略信息。和Glance一样，是template-based，里面包含app图标、app名称和通知标题。系统使用app的key color显示app名称：  
{% img left /images/blog/apple_watch_guidelines/4.shortlook_calendar_2x.png 488 337 %}  
通知标题可用的空间是相当小的，所以标题应该尽量简短，用于向用户简要描述此通知是做什么的。  

### Custom Long Look Notifications

Long Looks提供通知的详细信息。App可以自行定制Long Look的视图，也可以使用系统提供的默认视图。所有App的Long Look结构都是一样的。系统将会把app icon和app名称显示在顶部的sash area，sash area会给其下层的内容增加模糊效果。在通知的最底部是action按钮，app可以增加自定义action按钮，系统将会添加Dismiss按钮。在sash area和底部action按钮之间的中间部分，用于展示通知内容。  

{% img left /images/blog/apple_watch_guidelines/5.longlook_calendar_2x.png 479 703 %}  

关于Long Look Notifications：  

1. 通知内容可以垫在sash area的下层（underlap），透过半透明blur效果显示出来，也可以紧贴着显示在sash area的下面(just below)。一般来说图片可以选择垫在下层，而文字要紧贴着显示在sash area下面。  
{% img left /images/blog/apple_watch_guidelines/6.notification_long_content_2x.png 471 457 %}  
App需要定制Long Look显示的话, 必须提供static interface，也可一并提供dynamic interface。Static interface和dynamic interface都显示相同的通知类型。Dynamic interface比static interface定制性更强，而当dynamic interface不可用时，static interface提供了备用方案（fallback position）。关于static interface和dynamic interface在[Apple Watch Programming Guide](https://developer.apple.com/library/prerelease/ios/documentation/General/Conceptual/WatchKitProgrammingGuide/index.html#//apple_ref/doc/uid/TP40014969)中有详细介绍。  
2. Dismiss按钮将会始终显示。  
3. 除dismiss按钮之外，还可另外显示最多四个自定义action按钮。首先App向系统注册交互通知，然后Apple Watch根据注册的交互通知的类型来显示不同的action按钮。如何显示action自定义按钮在[Apple Watch Programming Guide](https://developer.apple.com/library/prerelease/ios/documentation/General/Conceptual/WatchKitProgrammingGuide/index.html#//apple_ref/doc/uid/TP40014969)中有详细介绍。  
4. Sash area的颜色和透明度是可以定制的。  

关于static and dynamic interfaces, 以及如何设置action按钮，移步[Apple Watch Programming Guide](https://developer.apple.com/library/prerelease/ios/documentation/General/Conceptual/WatchKitProgrammingGuide/index.html#//apple_ref/doc/uid/TP40014969)。

## Modal Sheets

{% img left /images/blog/apple_watch_guidelines/7.modal_interfaces_2x.jpg 547 343 %}  

Model sheet是一种弹出的视图，可以包含一屏子视图，也可以使用page-based布局包含多屏子视图。弹出时会临时阻挡软件其他部分的交互。  
一般用于：  

1. 让用户完成某个task；  
2. 向用户展示信息并引起用户的注意；  
3. 让用户继续选择子选项。  

App中应当尽量减少modal sheet的使用次数。一般来说，仅在满足下列条件时（二者皆满足或二者之一被满足）使用：  

1. 弹出的内容对于获取用户的注意是至关重要的；  
2. 包含的task必须被完成或隐式被禁用，以便避免用户数据处于模糊状态。  

注意事项：  

1. 左上角的关闭按钮是不能隐藏的，但是可以更改文案。但功能还是关闭作用。  
2. 如果需要提供给用户'接受'操作，请在modal sheet试图body上增加一个标准按钮。  
3. Modal task应该尽量简单，且避免从一个modal sheet进入另一个modal sheet。

## Layout

### Layout指南

1. 减少并列排放的控件数量。当按钮需要并列排放时，应当使用图标代替文字贴在按钮上。永远不要让三个以上的控件并列排放，不然按钮太小，用户很难点击到。  
2. 所有控件应该贴紧屏幕左右边缘显示，在物理机上，屏幕已在app内容外留有间距。注意这一点在模拟器上是看不到的。  
{% img left /images/blog/apple_watch_guidelines/8.fullwidth_settings_2x.png 272 700 %}  
3. 为控件使用相对定位，因为手表屏幕有不同的尺寸。  
4. 控件尽量使用从上到下、从左到右的对齐方式。  
5. 文字按钮宽度应该使用最长宽度。这样按钮上的文字才是始终可见的。  
6. 对于使用偏少的操作，使用上下文菜单来代替增加按钮，毕竟屏幕太小。  
{% img left /images/blog/apple_watch_guidelines/9.menu_stopwatch_2x.png 272 387 %}  

### 屏幕尺寸

1. 不管大屏幕还是小屏幕，应当使用相同的UI，然后自然拉伸或扩展。
{% img left /images/blog/apple_watch_guidelines/10.watch_screen_sizes_2x.png 637 454 %}  
2. 如果一张资源图可以兼容两种屏幕尺寸，那么准备一张就可以了，比如小的按钮。如果无法兼容两种屏幕尺寸，那么准备两张，比如填满屏幕的表盘背景。  

## Color and Typography

## Animations

## Branding


# 控件介绍

# 图片与图片的设计规范



