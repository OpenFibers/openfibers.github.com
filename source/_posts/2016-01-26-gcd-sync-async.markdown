---
layout: post
title: "GCD异步同步互转"
date: 2016-01-26 17:32:35 +0800
comments: true
categories: ['iOS', 'OSX', 'GCD']
---

## 同步转异步

这个是GCD最常见的用法之一，耗时长的操作放在global queue中做，完成之后将结果交给主线程处理：  

```objective-c
dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_BACKGROUND, 0), ^{
	id result = [self longTimeTask];
	dispatch_async(dispatch_get_main_queue(), ^{
		[self handleResultOnMainThread:result];
	});
});
```

## 异步转同步

这个本人并不常用，偶尔有不得不用的场景。  
在异步操作开始前create信号量，然后使用dispatch_semaphore_wait等待此信号量。在异步操作完成后对此信号量发送signal:  
```objective-c
dispatch_semaphore_t sema = dispatch_semaphore_create(0);
[self someAsyncTaskWithCallback:^(BOOL successed) {
   	dispatch_semaphore_signal(sema);
}];
dispatch_semaphore_wait(sema, DISPATCH_TIME_FOREVER);
```

Over