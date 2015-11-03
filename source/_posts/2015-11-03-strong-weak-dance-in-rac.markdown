---
layout: post
title: "Strong-weak dance in RAC"
date: 2015-11-03 10:12:32 +0800
comments: true
categories: ['iOS', 'RAC']
---

今天在工程中发现了RAC导致的retain cycle:  

```
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

 Over