---
layout: post
title: "不带导航条的UIViewController推出带导航条的UIViewController"
date: 2015-12-15 17:55:19 +0800
comments: true
categories: ['iOS', 'UIKit']
---

左侧controller A隐藏导航条，推出右侧controller B显示导航条，类似这个效果：  

{% img left /images/blog/push_navigation_bar/push_navigation_bar1.JPG 375 667%}    

<!--more-->

实现步骤如下：  

#### 1. navigation controller中推入controller A，在controller A初始化阶段设置导航条隐藏：

```objective-c
- (void)viewDidLoad
{
    [super viewDidLoad];

    ...
    self.navigationController.navigationBarHidden = YES;
}
```

#### 2. controller B在viewWillAppear时设置navigationBar显示，viewWillDisappear中设置navigationBar隐藏，animated要为YES

```
- (void)viewWillAppear:(BOOL)animated
{
    [super viewWillAppear:animated];
    [self.navigationController setNavigationBarHidden:NO animated:YES];
}

- (void)viewWillDisappear:(BOOL)animated
{
    [super viewWillDisappear:animated];
	[self.navigationController setNavigationBarHidden:YES animated:YES];
}
```

#### 3.如果controller B需要退出其他页面显示导航条，还需自行标记，使仅在back时切换显示/隐藏导航条

下面代码增加了pushingViewController标记，记录是否在推入下一级页面。非推入下一级页面的viewWillDisappear即为back，此时切换导航条为隐藏：  

```
- (void)viewWillAppear:(BOOL)animated
{
    [super viewWillAppear:animated];
    self.pushingViewController = NO;//显示前重置pushingViewController标记
    [self.navigationController setNavigationBarHidden:NO animated:YES];
}

- (void)viewWillDisappear:(BOOL)animated
{
    [super viewWillDisappear:animated];
    
    if (!self.pushingViewController)
    {
    	//不是推出下一级的viewWillDisappear（即back时）隐藏navigationBar
        [self.navigationController setNavigationBarHidden:YES animated:YES];
    }
}

- (void)pushOtherController
{
	self.pushViewController = YES;//标记正在推入下一级页面
    [self.navigationController pushViewController:viewController animated:YES];
}

```

Over  