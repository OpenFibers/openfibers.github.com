---
layout: post
title: "单页面隐藏 Navigation Bar，同时保持右滑手势退出页面的终极方案"
date: 2019-08-19 15:15:43 +0800
comments: true
categories: ['iOS', 'objective-c']
---

### 1. 隐藏 navigation bar

在需要隐藏 navigation bar 的 view controller 中：  

```objective-c

- (void)viewWillAppear:(BOOL)animated
{
    [super viewWillAppear:animated];
    [self hideNavBar];
}

- (void)viewDidDisappear:(BOOL)animated
{
    [super viewDidDisappear:animated];
    [self showNavBar];
}

- (void)hideNavBar
{
    if (self.navigationController.viewControllers.lastObject == self) //避免推出下一个隐藏bar bar的vc过场动画展示了nav bar
    {
        [self.navigationController setNavigationBarHidden:YES animated:YES];
    }
}

- (void)showNavBar
{
    if (self.navigationController.viewControllers.lastObject == self) //避免推出下一个隐藏bar bar的vc过场动画展示了nav bar
    {
        [self.navigationController setNavigationBarHidden:NO animated:YES];
    }
}

```

<!--more-->


### 2. 把手势搞出来

新增两个类：  

1) OTInteractivePopRecognizerDelegate:  

```
@interface OTInteractivePopRecognizerDelegate : NSObject <UIGestureRecognizerDelegate>
@property (nonatomic, weak) UINavigationController *navigationController;
@end

@implementation OTInteractivePopRecognizerDelegate

- (BOOL)gestureRecognizerShouldBegin:(UIGestureRecognizer *)gestureRecognizer
{
    return self.navigationController.viewControllers.count > 1;
}

- (BOOL)gestureRecognizer:(UIGestureRecognizer *)gestureRecognizer shouldRecognizeSimultaneouslyWithGestureRecognizer:(UIGestureRecognizer *)otherGestureRecognizer
{
    return YES;
}

@end
```

2) OTNavigationPopGestureUtil

```
#import <objc/runtime.h>

@interface OTNavigationPopGestureUtil : NSObject

+ (void)protectNavigationController:(UINavigationController *)navigationController;

@end

@implementation OTNavigationPopGestureUtil

+ (void)protectNavigationController:(UINavigationController *)navigationController
{
    OTInteractivePopRecognizerDelegate *popRecognizer = objc_getAssociatedObject(navigationController, _cmd);
    if (!popRecognizer)
    {
        popRecognizer = [[OTInteractivePopRecognizerDelegate alloc] init];
        popRecognizer.navigationController = navigationController;
        navigationController.interactivePopGestureRecognizer.delegate = popRecognizer;
        objc_setAssociatedObject(navigationController, _cmd, popRecognizer, OBJC_ASSOCIATION_RETAIN_NONATOMIC);
    }
}

@end


```

在需要隐藏 navigation bar 的 view controller 中：  

```
- (void)viewDidLoad
{
    [super viewDidLoad];
    [OTNavigationPopGestureUtil protectNavigationController:self.navigationController];
}
```

Over