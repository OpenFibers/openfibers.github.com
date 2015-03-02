---
layout: post
title: "In-App Purchase Walk Through"
date: 2015-02-28 16:19:15 +0800
comments: true
categories: [iOS]
---

# 适用情况

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

#购买及发放虚拟物品流程

官方给出的流程图是这样的：

![](https://developer.apple.com/library/ios/documentation/NetworkingInternet/Conceptual/StoreKitGuide/Art/intro_2x.png)

1. 获取内购列表（从App内读取或从自己服务器读取）
2. App Store请求可用的内购列表
3. 向用户展示内购列表
4. 用户选择了内购列表，再发个购买请求
5. 收到购买完成的回掉
6. 发放虚拟物品

# 虚拟物品

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

### 关于自动更新订阅品更新周期组（Auto-Renewable Subscription Duration Families）：  
每种订阅品的每种更新周期可以在iTunes Connect中设置一个单独的价格。图中给出了一种订阅品的不同长度的更新周期的价格截图：  
![](https://developer.apple.com/library/ios/documentation/LanguagesUtilities/Conceptual/iTunesConnectInAppPurchase_Guide/Art/iap_news_alerts_2x.png)
你可以把每种订阅品的每个长度的更新周期看成一个单独的产品，每个产品有自己独有的时长、价格、市场促销属性。  
为了让用户可以从中挑选一个偏好的更新周期，上图中我们为此种订阅的每个长度的更新周期分别设值了一个单独的价格，有一周的、一个月的、两个月的、季度的、半年的和一年的。  
上图中这种订阅品的全部六种更新周期合起来称为一个自动更新订阅品更新周期组（Auto-Renewable Subscription Duration Families）。  

# 人肉和iTunes Connect交互

### 新建虚拟物品
1. 登录[iTunes Connect](https://itunesconnect.apple.com/)
2. 点击My Apps
3. 进入想使用IAP的App详情
4. 选择In-App Purchases  
![](https://developer.apple.com/library/ios/documentation/LanguagesUtilities/Conceptual/iTunesConnectInAppPurchase_Guide/Art/AppDetails-menu-4_2x.png)
5. 点击Create New  
![](https://developer.apple.com/library/ios/documentation/LanguagesUtilities/Conceptual/iTunesConnectInAppPurchase_Guide/Art/In-AppPurchasesCreateNew_2x.png)
6. 选择IAP虚拟物品类型。注意虚拟物品一旦新建，类型无法修改。
7. 填入Internal Name。只能在iTunes Connect中看到这个名字。不会出现在App Store中。最长255字节。
8. 填入***Product ID***。每件物品有一个单独的***Product ID***，***Product ID***用于从App Store获取价格信息，以及付费时标识是哪种物品被购买了。例如com.163.neteasemusic.skin.dog。这个新建之后也是不能修改的。
9. 然后是设置价格。Cleared For Sale选为YES是虚拟物品被审核通过自动上架。NO是手动上架。Price Tier则是价格。
10. 多语言描述，这个是给用户看的。
11. 是否要在iTunes托管可购买内容，这个后面再说
12. 填入review notes和review用的截图。
13. 保存。
14. 保存后除了类型和Product ID都可以修改

# 代码里该做的事情

### 获取物品列表

SKProductRequest

### 发送购买请求

SKMutablePayment

### 观察购买状态

[[SKPaymentQueue defaultQueue] addTransactionObserver:observer];

### 二次验证是否为合法购买，没被破解

https://sandbox.itunes.apple.com/verifyReceipt
https://buy.itunes.apple.com/verifyReceipt