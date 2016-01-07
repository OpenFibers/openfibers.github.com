---
layout: post
title: "self在执行方法的过程中dealloc引起的崩溃处理"
date: 2016-01-07 11:27:54 +0800
comments: true
categories: 
---

### 场景和产生原因

在我的一个view controller里（为方便我们成为controller A），有一个doSomething方法，每次调用时，会延迟5秒调用delayAction，如果在5秒之内再次调用，会取消上次的延迟调用delayAction，再在5秒之后调用delayAction。  

这在UI操作中是一个稀松平常的需求，代码如下：  

```objective-c
- (void)doSomething
{
    [NSObject cancelPreviousPerformRequestsWithTarget:self selector:@selector(delayAction) object:nil];
    [self performSelector:@selector(delayAction) withObject:nil afterDelay:5];
}
```

然而就在某种情况下，这个正常工作的代码崩溃了，原因是第4行代码self已经释放，成为了僵尸指针，调用performSelector时造成了崩溃。  

<!--more-->

查了下原因，情况如下：  

#### 1. 
最开始controller A由其上一层的controller B保持了强引用：  
```
controller B -----> controller A
```

#### 2. 
第一次调用doSomething的时候，**performSelector:withObject:afterDalay:**使得runloop对controller A增加了一条强引用：  
```
controller B -----> controller A  
                         ^  
                         |  
runloop   ----------------  
```

#### 3. 
在某些操作之后，controller B dismiss了 controller A，于是B不再对A持有强引用，但由于**performSelector:withObject:afterDalay:**还未触发，runloop还对controller A保持了强引用，controller A还在存活：  

```
controller B - x -> controller A  
                         ^  
                         |  
runloop   ----------------  
```

#### 4. 
由于controller A还活着，在某些情况下又进入了doSomething，在第3行**cancelPreviousPerformRequestsWithTarget:selector:object:**时，runloop释放了对controller A的强引用，此时controller A dealloc了：  

```
controller B - x -> controller A (dealloced)  
                         ^  
                         x  
runloop   ----------------  
```

#### 5. 
controller A dealloc后，调用了第四行代码，向一个zombie的self指针发送了**performSelector:withObject:afterDelay**消息，然后崩溃了。当然，此时不单是调用self的performSelector方法，调用其他任何方法也会造成崩溃。  


### 解决方案

如[Self deallocs after cancelPreviousPerformRequestsWithTarget](http://stackoverflow.com/questions/15146235/self-deallocs-after-cancelpreviousperformrequestswithtarget)中所讨论，有两种方法：  

#### 第一种：retain-release dance: 

```
CFRetain((__bridge CFTypeRef)(self));
[NSObject cancelPreviousPerformRequestsWithTarget:self 
                                         selector:@selector(delayAction)
                                           object:nil];
[self delayAction];
CFRelease((__bridge CFTypeRef)(self));
```

先强行retain，再**cancelPreviousPerformRequestsWithTarget:selector:object:**，然后**delayAction**，最后强行release。  
这种方法保证self在delayAction的时候不会释放，保证delayAction操作可以继续做完。  

#### 第二种：weak指针做后面的delayAction，我把这种方式起名为philosophy dance（哲学的dance）

```
__weak typeof (self) (weakSelf) = self;
[NSObject cancelPreviousPerformRequestsWithTarget:self 
                                         selector:@selector(delayAction) 
                                           object:nil];
[weakSelf delayAction];
```

此法保证self在dealloc后，weakSelf为空，空指针调用**delayAction**则是安全的。优点是此时不会再耗费CPU时间进行**delayAction**的操作。  

以上两种做法在实际工程中，还需按照业务需求选取适合的。  

Over