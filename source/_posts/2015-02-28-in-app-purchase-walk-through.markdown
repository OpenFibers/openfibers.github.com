---
layout: post
title: "In-App Purchase Walk Through"
date: 2015-02-28 16:19:15 +0800
comments: true
categories: [iOS]
---

### 适用情况

想使用In-App Purchase（以下简称IAP）完成App内付费前，先确定需求是不是能用这个方案来满足。  
除了IAP以外，还有支付宝SDK、信用卡等其他方式完成软件内付费。  

在苹果制定的游戏规则中，所有在App内提供的服务需要付费时，都应当使用IAP，比如软件功能、游戏道具；所有在App外提供的服务需要付费时，都应使用其他支付方式，比如Uber的信用卡，淘宝、快的打车的支付宝SDK等。  

在IAP里，可以出售：  

1. 数字内容：比如杂志、图片、游戏关卡解锁、相机付费滤镜等；  
2. 软件功能：如各种扩展features；
3. 一次性服务：比如一次语音通话等。注意是「一次性」服务，不是一次「性服务」。

在IAP里，不能出售:  

1. 现实世界的商品或服务：比如刚才提到的一次「性服务」。  
2. 其他[the App Review GuideLines](https://developer.apple.com/appstore/guidelines.html)中规定的不允许的内容：比如刚才提到的一次「性服务」。

顺便说下，有次大网易的同事分享时提到：使用兑换码兑换App内服务是一条高压线。像Uber和Amazon里允许有码，是因为他们的码是用在现实世界的产品或服务上的。  

如果你确定内购需求符合IAP的使用要求，可以继续往下读了。  

<!--more-->

###购买及发放虚拟物品流程

官方给出的流程图是这样的：

![](https://developer.apple.com/library/ios/documentation/NetworkingInternet/Conceptual/StoreKitGuide/Art/intro_2x.png)

1. 向App Store请求内购列表
2. 向用户展示内购列表
3. 用户选择了内购列表，再发个购买请求
4. 收到购买完成的回掉
5. 发放虚拟物品

实际上第一步的内购列表可以提前预置在客户端内，或放在自己的服务器上（可以合并请求加快访问速度，而且连App Store本身也不快）。

### 虚拟物品的分类

虚拟物品分为以下几种类型：  

1. 消耗品（Consumable products）：比如游戏内金币等。
2. 不可消耗品（Non-consumable products）：简单来说就是一次购买，终身可用（用户可随时从App Store restore）。
3. 自动更新订阅品（Auto-renewable subscriptions）：和不可消耗品的不同点是有失效时间。比如一整年的付费周刊。在这种模式下，开发者定期投递内容，用户在订阅期内随时可以访问这些内容。订阅快要过期时，系统将自动更新订阅（如果用户同意）。
4. 非自动更新订阅品（Non-renewable subscriptions）：一般使用场景是从用户从IAP购买后，购买信息存放在自己的开发者服务器上。失效日期/可用是由开发者服务器自行控制的，而非由App Store控制，这一点与自动更新订阅品有差异。
5. 免费订阅品（Free subscriptions）：在Newsstand中放置免费订阅的一种方式。免费订阅永不过期。只能用于Newsstand-enabled apps。

类型2、3、5都是以Apple ID为粒度的。比如小张有三个iPad，有一个Apple ID购买了不可消耗品，则三个iPad上都可以使用。  
类型1、4一般来说则是现买现用。如果开发者自己想做更多控制，一般选4。

几种物品的区别如下（表格懒得翻译了）：

***Table 1-1  Comparison of product types***

Product type| Non-consumable | Consumable
-----------|-----------|-----------
Users can buy | Once | Multiple times
Appears in the receipt|Always|Once
Synced across devices|By the system|Not synced
Restored|By the system|Not restored

***Table 1-2  Comparison of subscription types***

Subscription type|Auto-renewable|Non-renewing|Free
--------|--------|--------|--------
Users can buy|Multiple times|Multiple times|Once
Appears in the receipt|Always|Once|Always
Synced across devices|By the system|By your app|By the system
Restored|By the system|By your app|By the system

### 在iTunes Connect上新建虚拟物品
每件物品有一个单独的***product identifier***，***product identifier***用于从App Store获取价格信息，以及付费时标识是哪种物品被购买了。

### 获取物品列表

SKProductRequest

### 发送购买请求

SKMutablePayment

### 观察购买状态

[[SKPaymentQueue defaultQueue] addTransactionObserver:observer];

### 二次验证是否为合法购买，没被破解

https://sandbox.itunes.apple.com/verifyReceipt
https://buy.itunes.apple.com/verifyReceipt