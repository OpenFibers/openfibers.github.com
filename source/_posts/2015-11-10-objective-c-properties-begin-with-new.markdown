---
layout: post
title: "objective-c中property以new开头报错"
date: 2015-11-10 15:11:27 +0800
comments: true
categories: ['objective-c','iOS','OSX']
---

在ARC中，属性名使用new开头会报错。比如说更改密码中原始密码的输入框叫`oldPasswordTextField`，可以；新密码的输入框叫`newPasswordTextField`，对不起，不行，编译错误。  

在ARC出现之前，方法名如果以alloc、copy、init、mutableCopy和new开头，标准做法是返回一个retain count+1的对象。在ARC中，默认声明时强制遵守了这一规范：方法名如果以alloc、copy、init、mutableCopy和new开头，会被隐式声明为attribute((ns_returns_retained))。此行为可被显示声明attribute((ns_returns_not_retained))覆盖。  

一般情况下不要使用attribute((ns_returns_not_retained))更改这一行为，除非有什么不得不履行的大义。  

引用[Clang 3.5 documentation | Objective-C Automatic Reference Counting | Retained return values](http://clang.llvm.org/docs/AutomaticReferenceCounting.html#retained-return-values):  

> A function or method which returns a retainable object pointer type may be marked as returning a retained value, signifying that the caller expects to take ownership of a +1 retain count.
> 
> …
> 
> Methods in the alloc, copy, init, mutableCopy, and new families are implicitly marked attribute((ns_returns_retained)). This may be suppressed by explicitly marking the method attribute((ns_returns_not_retained)).


参考帖子：[http://stackoverflow.com/questions/24308162/property-name-starting-with-new-prefix-leads-to-bad-access-error-in-ios](http://stackoverflow.com/questions/24308162/property-name-starting-with-new-prefix-leads-to-bad-access-error-in-ios)  

Over