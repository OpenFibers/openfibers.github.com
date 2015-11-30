---
layout: post
title: "Strong-weak dance 错误两则"
date: 2015-11-03 10:12:32 +0800
comments: true
categories: ['iOS', 'RAC']
---

## 第一则：RAC中strong-weak dance不完整造成内存泄露

今天在工程中发现了RAC导致的retain cycle:  

```objective-c
@weakify(self)
[RACObserve(self, fooProperty) subscribeNext:^(id fooProperty) {
    [self doSomething];
}];
```

相对于正常的RAC用法:  

```
@weakify(self)
[RACObserve(self, fooProperty) subscribeNext:^(id fooProperty) {
    @strongify(self)
    [self doSomething];
}];
```

少了一行**@strongify(self)**即造成了循环引用，即对于RAC来说，strong-weak dance是必须做的，不做strong-weak dance就会循环引用。  

<!--more-->

### 1. 为何RAC以外的block可以不使用strong-weak dance

然而对于「传统」的strong-weak dance来说，如果不需避免block体执行过程中self中途变空，不strong回来也是非常正常的做法，换言之，strong-weak dance是保证block执行过程中weakSelf不变空的一种技巧。比如：  

```
//不需避免block体执行过程中weakSelf中途变空，可以这样用
__weak id weakSelf = self;
[RACObserve(self, fooProperty) subscribeNext:^(id fooProperty) {
    [weakSelf doSomething];
}];
```

如需避免block体执行过程中weakSelf中途变空，则可以这样：  
```
//这样可以避免block体执行过程中weakSelf中途变空
__weak id weakSelf = self;
[RACObserve(self, fooProperty) subscribeNext:^(id fooProperty) {
    id strongSelf = weakSelf;
    [strongSelf doSomething];
}];
```

### 2. 为何RAC中不用strong-weak dance就会内存泄露

参考该贴：[Explanation of how weakify and strongify work in ReactiveCocoa / libextobjc](http://stackoverflow.com/questions/21716982/explanation-of-how-weakify-and-strongify-work-in-reactivecocoa-libextobjc)  

在预处理前的RAC代码：

```
@weakify(self)
[[self.searchText.rac_textSignal
    map:^id(NSString *text) {
        return [UIColor yellowColor];
    }]
    subscribeNext:^(UIColor *color) {
        @strongify(self)
        self.searchText.backgroundColor = color;
    }];
```

预处理后的RAC代码：

```
@autoreleasepool {} __attribute__((objc_ownership(weak))) __typeof__(self) self_weak_ = (self);
[[self.searchText.rac_textSignal
    map:^id(NSString *text) {
        return [UIColor yellowColor];
    }]
    subscribeNext:^(UIColor *color) {
        @try {} @finally {}
        __attribute__((objc_ownership(strong))) __typeof__(self) self = self_weak_; // 1
        self.searchText.backgroundColor = color;  //2
    }];
```

可以看到**@weakify**在block外声明了一个weak的**self_weak_**，赋值为标准意义的**self**，**@strongify**在block内声明了一个strong的名为**self**的变量，retain了**self_weak_**，覆盖了标准意义的**self**，于是在block内直接使用**self**实际是使用的**strong __typeof__(self) self**，并不是使用的标准意义的**self**，完成了完整的strong-weak dance。  
 而删除**@strongify(self)**，在block内则直接使用了标准意义的**self**，造成循环引用。  

## 第二则 在dealloc中使用strong-weak dance造成崩溃

此类崩溃其实很容易定位，只要运行到，崩溃是必现的。只怕业务逻辑使得有此dealloc不是每次运行都会进，再加上有稍懒同事提交了以下代码且没测试，就会造成崩溃：  

```
- (void)dealloc
{
    __weak typeof (self) weakSelf = self;//此时就会崩溃，已进入dealloc的对象不能再赋给weak指针了
    [self action:^{
        [weakSelf doSomeThing];
    }];
}
```

模拟器中运行，第三行报错EXC_BAD_INSTRUCTIONS(code=EXC_I386_INVOP)：  

> FooClass dealloced
> objc[16870]: Cannot form weak reference to instance (0x7fc02081ae50) of class FooClass. It is possible that this object was over-released, or is in the process of deallocation.  

额外的两点提示：  
1. dealloc中，如果不百分之百确定代码的行为，不要对内存做玩火的操作。  
2. 想开发一款卓越产品，在开发过程中须建立机制，保证每位开发人员运行测试了他所写的每一行代码。  

Over