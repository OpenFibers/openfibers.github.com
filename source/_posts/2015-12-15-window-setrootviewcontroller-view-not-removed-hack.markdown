---
layout: post
title: "[UIWindow setRootViewController:]后view无限叠加的问题修复"
date: 2015-12-15 17:22:34 +0800
comments: true
categories: ['iOS']
---

工程中有时会直接修改window.rootViewController，来导航到新的页面。  
按理说对window的rootViewController属性设好了新值，老的rootViewController被释放了，UIKit应当自动把老的rootViewController的view一并remove掉，然而实测并非如此。  
在iOS8/9中（更老版本没有测试），当window有rootViewController时，把新的controller赋值给window.rootViewController，老的rootViewController的view还是会留在window上。这些被遗留的view虽然看不见，但是浪费了内存，造成了view泄露；而且view的controller已经dealloc，此时view如果回调或通知controller的话，有造成崩溃的隐患。  

多次设置rootViewController后，view结构如图，window上加了多个UILayoutContainerView:  

{% img left /images/blog/window_setrootviewcontroller/window_setrootviewcontroller1.JPG %}  

<!--more-->

Stack Overflow上也有多个对于此问题的讨论，比如[UIWindow setRootViewController not clearing existing hierarchy](http://stackoverflow.com/questions/26795825/uiwindow-setrootviewcontroller-not-clearing-existing-hierarchy) 和 [Leaking views when changing rootViewController inside transitionWithView](http://stackoverflow.com/questions/26763020/leaking-views-when-changing-rootviewcontroller-inside-transitionwithview).  

实践了一下，因为工程中root window是继承自UIWindow的子类，所以直接重写了此类的`setRootViewController:`方法：  

```objective-c
//hack of setRootViewController: old rootViewController's view never removed from window
- (void)setRootViewController:(UIViewController *)rootViewController
{
    //remove old rootViewController's sub views
    for (UIView* subView in self.rootViewController.view.subviews)
    {
        [subView removeFromSuperview];
    }
    
    //remove old rootViewController's view
    [self.rootViewController.view removeFromSuperview];
    
    //set new rootViewController
    [super setRootViewController:rootViewController];
    
    //remove empty UILayoutContainerView(s) remaining on root window
    for (UIView *subView in self.subviews)
    {
        if (subView.subviews.count == 0)
        {
            [subView removeFromSuperview];
        }
    }
}
```

Over  