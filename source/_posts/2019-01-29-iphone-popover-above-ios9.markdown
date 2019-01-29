---
layout: post
title: "iPhone Popover above iOS9"
date: 2019-01-29 16:44:13 +0800
comments: true
categories: 
---

直接上图上代码。。。

{% img left /images/blog/iphone_popover/iphone_popover.jpg %}  

OTPopoverMenuViewController.h:  
```
@interface OTPopoverMenuViewController : UIViewController

- (void)presentInController:(UIViewController *)controller
                 sourceView:(UIView *)view;

- (void)presentInController:(UIViewController *)controller
              barButtonItem:(UIBarButtonItem *)barButtonItem;


@end
```

<!--more-->

OTPopoverMenuViewController.m:  
```
#import "OTPopoverMenuViewController.h"

@interface OTPopoverMenuViewController () <UIPopoverPresentationControllerDelegate>

@end

@implementation OTPopoverMenuViewController

- (instancetype)initWithNibName:(NSString *)nibNameOrNil bundle:(NSBundle *)nibBundleOrNil
{
    self = [super initWithNibName:nibNameOrNil bundle:nibBundleOrNil];
    if (self)
    {
        self.modalPresentationStyle = UIModalPresentationPopover;
        
        UIPopoverPresentationController *popController = [self popoverPresentationController];
        self.preferredContentSize = CGSizeMake(100, 100);
        popController.permittedArrowDirections = UIPopoverArrowDirectionUp;
        popController.delegate = self;
        
        [self initUI];
    }
    return self;
}

- (void)presentInController:(UIViewController *)controller
                 sourceView:(UIView *)view
{
    UIPopoverPresentationController *popController = [self popoverPresentationController];
    popController.sourceView = view;
    popController.sourceRect = view.bounds;
    [controller presentViewController:self animated:YES completion:nil];
}

- (void)presentInController:(UIViewController *)controller
              barButtonItem:(UIBarButtonItem *)barButtonItem
{
    UIPopoverPresentationController *popController = [self popoverPresentationController];
    popController.barButtonItem = barButtonItem;
    [controller presentViewController:self animated:YES completion:nil];
}

- (UIModalPresentationStyle)adaptivePresentationStyleForPresentationController:(UIPresentationController *)controller
{
    return UIModalPresentationNone;
}

- (void)initUI
{
    
}

@end
```


调用:  

```
OTPopoverMenuViewController *controller = [[OTPopoverMenuViewController alloc] init];
[controller presentInController:self sourceView:view];
//或
[controller presentInController:self barButtonItem:barButtonItem];
```

Over