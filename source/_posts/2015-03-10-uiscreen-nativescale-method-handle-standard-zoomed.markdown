---
layout: post
title: "处理iPhone6、6+标准视图和放大视图 & 新增的[UIScreen nativeScale]方法"
date: 2015-03-10 14:08:47 +0800
comments: true
categories: ["iOS"]
---

### 标准模式与放大模式

iPhone6和6+的设置(Settings)->显示与亮度(Display & Brightness)->显示模式(View)都带有标准模式(Standard)和放大模式(Zoomed)。  
这个功能被引入后，依赖`[UIScreen bounds]`和`[UIScreen scale]`并不能完全确定屏幕分辨率是多大、用户选择了放大试图还是标准视图。比如放大模式下iPhone6读到的这两个属性和iPhone5是一模一样的，而放大模式下iPhone6+的`[UIScreen bounds]`属性和标准模式下iPhone6的一样。

iOS8中苹果引入了`[UIScreen screenScale]`，可以用来区分不同的显示模式。  

<!--more-->

### 关于[UIScreen screenScale]

官方文档给出的解释则非常惜字如金：

```
The native scale factor for the physical screen. (read-only)
```

何为`native scale factor`? 写文档这哥们有点懒啊，妥妥的中国大陆互联网公司文档范儿，搞不好是本着'不知道上层用户用得着用不着先开放了再说'的原则做的底层。猜测是物理分辨率与`[UIScreen bounds]`的比值。光猜不放心，最好还是做个实验看下这个猜测是不是对的。  

```objective-c
    CGRect bounds = [[UIScreen mainScreen] bounds];
    NSString *screenMode = [[UIScreen mainScreen].coordinateSpace description];
    CGFloat scale = [[UIScreen mainScreen] scale];
    CGFloat nativeScale = [[UIScreen mainScreen] nativeScale];
    
    NSLog(@"\n bounds: %@\n screen mode: %@\n scale: %f\n native scale: %f", NSStringFromCGRect(bounds), screenMode, scale, nativeScale);
```

iPhone6标准模式下输出：

```
ScreenTest[3441:3088752] 
 bounds: { {0, 0}, {375, 667} }
 screen mode: <UIScreen: 0x155603cd0; bounds = { {0, 0}, {375, 667} }; mode = <UIScreenMode: 0x170021f40; size = 750.000000 x 1334.000000>>
 scale: 2.000000
 native scale: 2.000000
```

iPhone6放大模式下输出：

```
ScreenTest[3454:3089937] 
 bounds: { {0, 0}, {320, 568} }
 screen mode: <UIScreen: 0x12ee03d40; bounds = { {0, 0}, {320, 568} }; mode = <UIScreenMode: 0x1700256e0; size = 640.000000 x 1136.000000>>
 scale: 2.000000
 native scale: 2.343750
```

2.343750是个啥？1134/568是2.34859154，比较接近了。而750/320就是2.343750。  

再来看iPhone6+标准模式下输出：

```
ScreenTest[1876:465146] 
 bounds: { {0, 0}, {414, 736} }
 screen mode: <UIScreen: 0x13ee01840; bounds = { {0, 0}, {414, 736} }; mode = <UIScreenMode: 0x17002b4e0; size = 1242.000000 x 2208.000000>>
 scale: 3.000000
 native scale: 2.608696
```

iPhone6+放大模式下输出：
```
ScreenTest[1893:466244] 
 bounds: { {0, 0}, {375, 667} }
 screen mode: <UIScreen: 0x13f6021f0; bounds = { {0, 0}, {375, 667} }; mode = <UIScreenMode: 0x170028c20; size = 1125.000000 x 2001.000000>>
 scale: 3.000000
 native scale: 2.880000
```

iPhone6+的物理分辨率是1080*1920，1080/414是2.608696，1080/375是2.880000。  
需要注意的是此接口从iOS8开始支持。代码里要用的话记得加个保护。  

### 苹果希望开发者们怎样做？
从文档来推断，UIKit的设计者可能觉得这个接口使用频率不会很高，以至于文档里只给出了一句不清晰的解释，没说明比值以宽度计算，也没给出其他建议。继续推断在UIKit的设计者认为`[UIScreen bounds]`和`[UIScreen scale]`才是写layout时主力的检测分辨率方法，无论是CoreGraphics还是图片scale选取，都应该以`[UIScreen scale]`为准。在iPhone6上，无论标准模式还是放大模式，scale始终是2，所以绘制和图片都应该选择2倍大小；iPhone6+上，无论标准模式还是放大模式，scale始终是3，所以绘制和图片都应该选择3倍大小。  

当然UIKit并不是开发UI的唯一方式，于是`[UIScreen nativeScale]`有了其存在的更多意义，尤其是做偏底层的绘制工作时。  

Over