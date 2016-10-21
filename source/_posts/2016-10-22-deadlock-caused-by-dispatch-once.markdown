---
layout: post
title: "dispatch_once造成的死锁----分析、解决与自动检测"
date: 2016-10-22 03:40:14 +0800
comments: true
categories: [iOS, macOS, runtime]
---


### 现象

最近遇到了一个死锁crash，主线程在dispatch_once时卡住了：  

```objective-c
Thread 0 name:  Dispatch queue: com.apple.main-thread
Thread 0 Crashed:
0   __ulock_wait + 8
1   _dispatch_unfair_lock_wait + 48
2   _dispatch_gate_wait_slow + 56
3   dispatch_once_f + 124
4   +[OTPolicyCenter sharedInstance] (once.h:68)
7   +[OTWebViewUtil completeUrlScheme:] (WVWebViewUtil.m:26)
...
30  start + 4
```

卡死的代码很简单，世界上的单例基本上都是这么开的：  

```
+ (OTPolicyCenter *)sharedInstance
{
    static dispatch_once_t onceToken;
    dispatch_once(&onceToken, ^{
        policyCenterInstance = [[OTPolicyCenter alloc] init];
    });
    return policyCenterInstance;
}
```

其他线程大多数也都卡住了（除了带runloop的线程和事情还没做完的线程）：  

```
Thread 1:
0   __psynch_cvwait + 8
1   _pthread_cond_wait + 640
2   -[__NSOperationInternal _waitUntilFinished:] + 132
3   -[__NSObserver _doit:] + 232
4   __CFNOTIFICATIONCENTER_IS_CALLING_OUT_TO_AN_OBSERVER__ + 20
5   _CFXRegistrationPost + 400
6   ___CFXNotificationPost_block_invoke + 60
7   -[_CFXNotificationRegistrar find:object:observer:enumerator:] + 1504
8   _CFXNotificationPost + 376
9   -[NSNotificationCenter postNotificationName:object:userInfo:] + 68
10  -[CTTelephonyNetworkInfo queryDataMode] + 408
11  -[CTTelephonyNetworkInfo init] + 336
12  -[OTReachability networkStatusForFlags:] (AFReachability.m:216)
...
25  start_wqthread + 4

Thread 4:
0   __semwait_signal + 8
1   nanosleep + 212
2   usleep + 64
3   wpthread_main + 216
4   _pthread_body + 240
5   _pthread_body + 0
6   thread_start + 4

Thread 7 name:  Dispatch queue: com.apple.root.default-qos
Thread 7:
0   semaphore_wait_trap + 8
1   _dispatch_semaphore_wait_slow + 216
2   CFURLConnectionSendSynchronousRequest + 284
3   +[NSURLConnection sendSynchronousRequest:returningResponse:error:] + 120
4   +[UTMCHttpHelper post:url:dict:len:errorCode:] + 1864
5   __39-[UTMCOnlineConfManager syncOnlineconf]_block_invoke + 660
6   _dispatch_call_block_and_release + 24
7   _dispatch_client_callout + 16
8   _dispatch_queue_override_invoke + 732
9   _dispatch_root_queue_drain + 572
10  _dispatch_worker_thread3 + 124
11  _pthread_wqthread + 1288
12  start_wqthread + 4

Thread 18 name:  Dispatch queue: com.apple.NSURLSession-work
Thread 18:
0   __psynch_cvwait + 8
1   _pthread_cond_wait + 640
2   -[__NSOperationInternal _waitUntilFinished:] + 132
3   -[__NSObserver _doit:] + 232
4   __CFNOTIFICATIONCENTER_IS_CALLING_OUT_TO_AN_OBSERVER__ + 20
5   _CFXRegistrationPost + 400
6   ___CFXNotificationPost_block_invoke + 60
7   -[_CFXNotificationRegistrar find:object:observer:enumerator:] + 1504
8   _CFXNotificationPost + 376
9   -[NSNotificationCenter postNotificationName:object:userInfo:] + 68
10  -[CTTelephonyNetworkInfo queryDataMode] + 408
11  -[CTTelephonyNetworkInfo init] + 336
12  -[OTPolicyCenter init] (NWPolicyCenter.m:52)
13  __31+[NWPolicyCenter sharedInstance]_block_invoke (NWPolicyCenter.m:43)
14  _dispatch_client_callout + 16
15  dispatch_once_f + 56
16  +[OTPolicyCenter sharedInstance] (once.h:68)
17  +[OTUtils singletonObject:getter:] (OTUtils.m:271)
...
43  start_wqthread + 4
```

### 原因

在主线程中卡死前的一行`[OTPolicyCenter sharedInstance]`，在线程18中也找到了相同的调用。      

再来看一眼这个简单的单例方法：    

```
+ (OTPolicyCenter *)sharedInstance
{
    static dispatch_once_t onceToken;
    dispatch_once(&onceToken, ^{
        policyCenterInstance = [[OTPolicyCenter alloc] init];
    });
    return policyCenterInstance;
}
```

线程18首先进入了`sharedInstance`，在`dispatch_once(&onceToken, ^)`时锁住了`onceToken`，主线程稍后进入`sharedInstance`，阻塞在`dispatch_once(&onceToken, ^)`这里，而线程18继续往下执行到`[[OTPolicyCenter alloc] init]`。  

此时线程18阻塞式的向主线程发出了操作：`[__NSOperationInternal _waitUntilFinished:]`。因为主线程在阻塞中等待`onceToken`，所以主线程不能接收线程18的通知，于是线程18一直在等主线程接受通知，也不会去释放`onceToken`，死锁生成。  

至于为什么`[NSNotificationCenter postNotificationName:object:userInfo:]`会同步等待主线程返回，猜测苹果自己在实现中接收通知是这样做的，要求接收通知的block在mainQueue上执行：  
```
[[NSNotificationCenter defaultCenter] 
  addObserverForName:NotificationName
              object:nil
               queue:[NSOperationQueue mainQueue]
          usingBlock:^(NSNotification *ns) {
              NSLog(@"Notification %@", ns);
}];
```

当然此时线程18上如果不是发了一个阻塞式的通知，而是做了一些其他的需要在主线程执行并同步返回的事，也会造成死锁。  

### 解决方案

1. 自动解决或加保护？  
如果围绕自动解决或者加保护的方式来做，禁止子线程同步调用主线程也好是不现实的（总有业务限制），禁止子线程和主线程共享单例也是不现实的（总有业务限制），所有单例串行执行可能会造成性能问题而且风险很大。目前还没想到可行的方案。  

2. 静态检测工具？  
首先要做静态分析，毕竟之前没做过，门槛太高，性价比低，放弃。  

3. 运行时检测工具？  
想做运行时检测有两件事要做：  
**第一件事，在线程申请加锁和解锁once token时，对线程打标记：**  
自己的代码中可以用宏定义改掉dispatch_once的实现，在其中对线程打标记，这个应该不难。  
别人的代码中只能在运行时里面换出sharedInstance, defaultManager等方法来打标记。  
**第二件事，找出子线程准备锁主线程的位置：**  
仅可以 hook objective-c 实现的同步方法，不能 hook GCD 的同步方法，所以仍要靠人肉review，而且只能review自己代码，不能review SDK。  
结论是制作此工具可以用作预检，减轻我们部分负担。有时间可以尝试写一下。  

4. 使用的时候人肉多加注意？  
为保证稳定不在dispatch_once中同步执行主线程任务。但是人肉保证难度大。  

结论是3和4可以尝试做一下。  

相关问题：[The Good, the Bad and the Notification](https://medium.com/@elfenlaid/the-good-the-bad-and-the-notification-275198fa2e86#.pyvxgndad)。

