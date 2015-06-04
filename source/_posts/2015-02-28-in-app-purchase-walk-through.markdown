---
layout: post
title: "In-App Purchase Walk Through"
date: 2015-02-28 16:19:15 +0800
comments: true
categories: ["iOS"]
---

## 1. 适用情况

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

## 2. 购买及发放虚拟产品流程

官方给出的流程图是这样的：

{% img left /images/blog/iap/1.iap_intro_2x.png %}

1. 获取内购列表（从App内读取或从自己服务器读取）
2. App Store请求可用的内购列表
3. 向用户展示内购列表
4. 用户选择了内购列表，再发个购买请求
5. 收到购买完成的回掉
6. 发放虚拟产品

## 3.虚拟产品

### 虚拟产品的分类
虚拟产品分为以下几种类型：  

1. 消耗品（Consumable products）：比如游戏内金币等。
2. 不可消耗品（Non-consumable products）：简单来说就是一次购买，终身可用（用户可随时从App Store restore）。
3. 自动更新订阅品（Auto-renewable subscriptions）：和不可消耗品的不同点是有失效时间。比如一整年的付费周刊。在这种模式下，开发者定期投递内容，用户在订阅期内随时可以访问这些内容。订阅快要过期时，系统将自动更新订阅（如果用户同意）。
4. 非自动更新订阅品（Non-renewable subscriptions）：一般使用场景是从用户从IAP购买后，购买信息存放在自己的开发者服务器上。失效日期/可用是由开发者服务器自行控制的，而非由App Store控制，这一点与自动更新订阅品有差异。
5. 免费订阅品（Free subscriptions）：在Newsstand中放置免费订阅的一种方式。免费订阅永不过期。只能用于Newsstand-enabled apps。

类型2、3、5都是以Apple ID为粒度的。比如小张有三个iPad，有一个Apple ID购买了不可消耗品，则三个iPad上都可以使用。  
类型1、4一般来说则是现买现用。如果开发者自己想做更多控制，一般选4。

几种产品的区别如下（表格懒得翻译了）：

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
{% img left /images/blog/iap/2.iap_news_alerts_2x.jpg %}  
你可以把每种订阅品的每个长度的更新周期看成一个单独的产品，每个产品有自己独有的时长、价格、市场促销属性。  
为了让用户可以从中挑选一个偏好的更新周期，上图中我们为此种订阅的每个长度的更新周期分别设值了一个单独的价格，有一周的、一个月的、两个月的、季度的、半年的和一年的。  
上图中这种订阅品的全部六种更新周期合起来称为一个自动更新订阅品更新周期组（Auto-Renewable Subscription Duration Families）。  

## 4. 人肉和iTunes Connect交互

### 填写银行卡与纳税信息
1. 登录[iTunes Connect](https://itunesconnect.apple.com/)
2. 点击Agreements, Tax, and Banking，填写Contact Info, Bank Info, Tax Info。如果填过就不用再填了。刚刚填写完成后，各种信息正在审核，如下图：  
{% img left /images/blog/iap/17.iap_tax_banking.png %}  
即使信息正在审核，沙箱环境下也是可以访问IAP服务的，并不需要等审核完成才能测试。  

### 新建虚拟产品
1. 登录[iTunes Connect](https://itunesconnect.apple.com/)
2. 点击My Apps
3. 进入想使用IAP的App详情
4. 选择In-App Purchases  
{% img left /images/blog/iap/3.iap_AppDetails-menu-4_2x.png %}
5. 点击Create New  
6. 选择IAP虚拟产品类型。注意虚拟产品一旦新建，类型无法修改。  
{% img left /images/blog/iap/5.iap_iTunes_SelectType.png %}
7. 填入Internal Name。只能在iTunes Connect中看到这个名字。不会出现在App Store中。最长255字节。  
{% img left /images/blog/iap/6.iap_internal_name.png %}
8. 填入***Product ID***。每件产品有一个单独的***Product ID***，***Product ID***用于从App Store获取价格信息，以及付费时标识是哪种产品被购买了。例如com.163.neteasemusic.skin.dog。这个新建之后也是不能修改的。  
{% img left /images/blog/iap/7.iap_product_id.png %}
9. 然后是设置价格。Cleared For Sale选为YES是虚拟产品被审核通过自动上架。NO是手动上架。Price Tier则是价格。  
{% img left /images/blog/iap/8.iap_price.png %}
10. 多语言描述，这个是给用户看的。  
{% img left /images/blog/iap/9.iap_add_language_1.png %}
{% img left /images/blog/iap/10.iap_add_language_2.png %}
11. 是否要在iTunes托管可购买内容，这个后面再说  
{% img left /images/blog/iap/11.iap_host_contents.png %}
12. 填入review notes和review用的截图。  
{% img left /images/blog/iap/12.iap_review_notes_and_screen_shot.png %}
13. 保存。
14. 保存后除了类型和Product ID都可以修改  
{% img left /images/blog/iap/13.iap_edit_product1.png %}
{% img left /images/blog/iap/14.iap_edit_product2.png %}
{% img left /images/blog/iap/15.iap_edit_product3.png %}
{% img left /images/blog/iap/16.iap_edit_product4.png %}

新建完，不用等待苹果审核就可以在沙箱环境使用了。  

### 新建测试帐号
1. 登录[iTunes Connect](https://itunesconnect.apple.com/)  
2. 点击Users and Roles  
3. 点击Sandbox Testers  
4. 添加Tester。添加后在测试机上用tester帐号登录app store  

### 附：在苹果托管不可消耗品（Non-consumable products）的内容需知
托管内容仅限于针对不可消耗品。  
首次创建不可消耗品时可以选择把内容托管到苹果服务器，当然也可以随时将自己服务器上的内容迁移到苹果服务器由苹果托管。  
需要使用托管功能的话，首先在iTunes Connect中提交不可消耗品让苹果审核。然后在Xcode中选取In-App Purchase Content template创建虚拟产品, 放入需要托管的内容, 然后使用Archive功能上传。或者使用Xcode为每一种虚拟产品创建一个.pkg文件，然后使用Application Loader一次性上传。  
具体细节请参考[Using Application Loader](https://itunesconnect.apple.com/docs/UsingApplicationLoader.pdf)中和In-App Purchase有关的章节。

关于和iTunes Connect的交互，更多细节请参考[In-App Purchase Configuration Guide for iTunes Connect](https://developer.apple.com/library/ios/documentation/LanguagesUtilities/Conceptual/iTunesConnectInAppPurchase_Guide/Chapters/Introduction.html#//apple_ref/doc/uid/TP40013727-CH1-SW1)。

## 5. 代码里该做的事情

### 获取产品列表

1. 首先读取出App中内嵌的或是服务端中的Product IDs。
2. 使用SKProductRequest向苹果服务器验证哪些Product IDs是可用的。注意不要在[Class load]里发送请求，一定要等到App didFinishLauching之后再发，不然无法接到请求返回。核心代码如下：
```objective-c
	#import <StoreKit/StoreKit.h>
	#define kInAppPurchaseProUpgradeProductId   @"com.163.neteasemusic.skin.dog"
	...
    NSSet *productIDs = [NSSet setWithObject:kInAppPurchaseProUpgradeProductId];
    SKProductsRequest *request= [[SKProductsRequest alloc] initWithProductIdentifiers:productIDs];
    request.delegate = self;
    [request start];
```

接收结果
```objective-c
- (void)productsRequest:(SKProductsRequest *)request
     didReceiveResponse:(SKProductsResponse *)response
{
    NSArray *myProducts = response.products;
    for (SKProduct *product in myProducts)
    {
        //product
    }
}

- (void)request:(SKRequest *)request didFailWithError:(NSError *)error
{
	//处理错误
}
```

### 向自己的服务器生成订单  
如果需要经过自己的服务器做二次验证，建议在调用苹果支付接口前做这一步。  
订单中必须要保存的是订单ID和用户想要购买的商品ID。这个记录是为了在二次验证时服务端做检查，防止 A 商品的 receipt 被用户拿来做 B 商品的购买结果校验。  

### 发送购买请求
```objective-c
	#import <StoreKit/StoreKit.h>
	...
	SKProduct *product = <# products request中返回的SKProduct #>;
	SKMutablePayment *payment = [SKMutablePayment paymentWithProduct:product];
	payment.quantity = 2;
	[[SKPaymentQueue defaultQueue] addPayment:payment];
```
或者
```objective-c
	#import <StoreKit/StoreKit.h>
	...
	SKProduct *product = <# products request中返回的SKProduct #>;
	SKPayment *payment = [SKPayment paymentWithProduct:product];
	[[SKPaymentQueue defaultQueue] addPayment:payment];
```

### 观察购买状态

首先在程序启动时注册观察者
```objective-c
	#import <StoreKit/StoreKit.h>
	...
	[[SKPaymentQueue defaultQueue] addTransactionObserver:observer];
```

并且实现回调，处理相应的购买返回。
```objective-c
- (void)paymentQueue:(SKPaymentQueue *)queue
 updatedTransactions:(NSArray *)transactions
{
    for (SKPaymentTransaction *transaction in transactions)
    {
        switch (transaction.transactionState)
        {
            // Call the appropriate custom method for the transaction state.
            case SKPaymentTransactionStatePurchasing:
                [self showTransactionAsInProgress:transaction deferred:NO];
                break;
            case SKPaymentTransactionStateDeferred:
                [self showTransactionAsInProgress:transaction deferred:YES];
                break;
            case SKPaymentTransactionStateFailed:
                [self failedTransaction:transaction];
                break;
            case SKPaymentTransactionStatePurchased:
                [self completeTransaction:transaction];
                break;
            case SKPaymentTransactionStateRestored:
                [self restoreTransaction:transaction];
                break;
            default:
                // For debugging
                NSLog(@"Unexpected transaction state %@", @(transaction.transactionState));
                break;
        }
    }
}
```

需要监听SKPaymentQueue的更多状态变更，请实现[SKPaymentTransactionObserver](https://developer.apple.com/library/ios/documentation/StoreKit/Reference/SKPaymentTransactionObserver_Protocol/index.html#//apple_ref/occ/intf/SKPaymentTransactionObserver)协议中提供的更多方法。

### 完成购买
在收到Purchased或Restored回调后，持久化购买记录以及receipt data。  
然后通知PaymentQueue，购买已经完成了。对finishTransaction则会触发系统IAP的UI刷新：
```objective-c
	SKPaymentTransaction *transaction = <# The current payment #>;
	[[SKPaymentQueue defaultQueue] finishTransaction:transaction];
```
另外在发放功能或道具之前，最好在自己服务端做一次二次校验，防止越狱插件或者Wifi的HTTP代理伪造购买记录。

### 二次验证防止破解

越狱插件或者HTTP代理均可让用户做到伪造购买记录。当我们收到购买完成的回调后，最好经过自己服务器验证购买是否合法。  

####经过 App Store 验证

以下代码用Cocoa实现了二次验证的过程。但是这个过程最好通过自己的后台服务器来做，不然非常容易在客户端被伪造返回结果。  
这里使用Cocoa实现只是为了阐述请求与返回值的格式。
发送二次验证请求：

```objective-c
#define SANDBOX_VERIFY_RECEIPT_URL          [NSURL URLWithString:@"https://sandbox.itunes.apple.com/verifyReceipt"]
#define APP_STORE_VERIFY_RECEIPT_URL        [NSURL URLWithString:@"https://buy.itunes.apple.com/verifyReceipt"]

#ifdef DEBUG
#define VERIFY_RECEIPT_URL SANDBOX_VERIFY_RECEIPT_URL
#else
#define VERIFY_RECEIPT_URL APP_STORE_VERIFY_RECEIPT_URL
#endif

...

- (void)verifyTransaction:(SKPaymentTransaction *)transaction
{
    NSData *transactionReceipt = transaction.transactionReceipt;
    NSString *base64String = [OTBase64Helper base64forData:transactionReceipt];
    NSDictionary *receiptDictionary = @{@"receipt-data":base64String};
    NSData *data = [receiptDictionary JSONData];
    
    if (_receiptRequest)
    {
        [_receiptRequest cancel];
        _receiptRequest = nil;
    }
    _receiptRequest = [[ASIFormDataRequest alloc] initWithURL:VERIFY_RECEIPT_URL];
    _receiptRequest.userInfo = @{@"ProductIdentifier" : transaction.payment.productIdentifier};
    _receiptRequest.delegate = self;
    [_receiptRequest appendPostData:data];
    [_receiptRequest startAsynchronous];
}
```

接收二次验证结果：
```objective-c
- (void)requestFinished:(ASIHTTPRequest *)request
{
    NSString *responseString = [request responseString];
    NSDictionary *dictionary = [responseString objectFromJSONString];
    NSString *productId = dictionary[@"receipt"][@"product_id"];
    NSNumber *status = dictionary[@"status"];
    if (status.intValue == 0)
    {
        //校验成功，发放内容
         //status code 0为成功
   }
    else
    {
        //校验失败，不做处理或相应惩罚
        //21000 App Store不能读取你提供的JSON对象
        //21002 receipt-data域的数据有问题
        //21003 receipt无法通过验证
        //21004 提供的shared secret不匹配你账号中的shared secret
        //21005 receipt服务器当前不可用
        //21006 receipt合法，但是订阅已过期。服务器接收到这个状态码时，receipt数据仍然会解码并一起发送
        //21007 receipt是Sandbox receipt，但却发送至生产系统的验证服务
        //21008 receipt是生产receipt，但却发送至Sandbox环境的验证服务
    }
}

- (void)requestFailed:(ASIHTTPRequest *)request
{
    //出错处理
}
```

苹果的返回值如下：  
```JSON
{
"receipt": {
            "original_purchase_date_pst":"2015-06-03 04:00:37 America/Los_Angeles",
            "purchase_date_ms":"1433329237329",
            "unique_identifier":"secret9f135e2cd8f7dda951a15c01cd2220c60b",
            "original_transaction_id":"1000000157783770",
            "bvrs":"2.6.0",
            "transaction_id":"1000000157783770",
            "quantity":"1",
            "unique_vendor_identifier":"SECRETCD-89AD-45C4-8937-359CCA9E8F36",
            "item_id":"SECRET509",
            "product_id":"com.your.iap.product.id",
            "purchase_date":"2015-06-03 11:00:37 Etc/GMT",
            "original_purchase_date":"2015-06-03 11:00:37 Etc/GMT",
            "purchase_date_pst":"2015-06-03 04:00:37 America/Los_Angeles",
            "bid":"com.your.app.bundle.id",
            "original_purchase_date_ms":"1433329237329"
            },
"status": 0
}
```

#### 纯本地验证
除了网络验证以外，苹果提供了纯粹的本地验证方式：[Validating Receipts Locally](https://developer.apple.com/library/ios/releasenotes/General/ValidateAppStoreReceipt/Chapters/ValidateLocally.html#//apple_ref/doc/uid/TP40010573-CH1-SW2).  
这种方式可以做到防止被通用破解方式破解，但并不能防止针对特定 App 的破解。  

#### 第三方网站验证
有些第三方网站提供了经服务端的验证服务。比如[urbanairship](http://urbanairship.com/products). 但是我并没有用过，所以不知道具体效果如何。毕竟第三方服务无法做到在用户发起购买之前生成订单记录，与购买后验证结果比对，所以我还是比较担心第三方验证服务的安全性的。而且鸡国网络连国外验证服务器，你懂的。。

总之想要万无一失，建议开发自己的验证接口。  

更多验证相关问题，请参考[Receipt Validation Programming Guide](https://developer.apple.com/library/ios/releasenotes/General/ValidateAppStoreReceipt/Introduction.html#//apple_ref/doc/uid/TP40010573)

验证成功后，则是真正的发放内容、道具等。  

## 6.服务端二次验证后再发放数据中的安全问题

由于是和钱关系最紧密的功能，IAP安全性显得无比重要。  
如果是用的“经服务端二次验证成功发放道具”的逻辑，而非“购买成功发放道具，二次验证失败惩罚处理”，则在我的实践过程中，以下几件事是必须要做的。  

### 客户端防止用户数据丢失
不像支付宝SDK那样全部校验在服务端做，用IAP时部分流程的完整性是需要客户端保证的。  
在transaction完成后，和服务端的二次验证完成前，要对transaction.transactionReceipt做持久化。  
删除此持久化的时机应当是收到从服务端发回的二次验证请求的响应时，确认服务端已和苹果完成通信之后（服务端返回和苹果连接失败则不应删除已保存的receiptData）。  

### 服务端防止被盗  
服务端防盗主要有两点：  
1. 自己服务器的线上环境避免使用苹果sandbox环境做二次验证，防止公司内部使用同一个apple developer id的人建立sandbox test user监守自盗。  
2. 对验证通过的小票做废弃记录，防止黑客使用同一个小票反复验证购买。  
再回顾一下，苹果二次验证接口的返回值如下：  

```JSON
{
"receipt": {
            "original_purchase_date_pst":"2015-06-03 04:00:37 America/Los_Angeles",
            "purchase_date_ms":"1433329237329",
            "unique_identifier":"secret9f135e2cd8f7dda951a15c01cd2220c60b",
            "original_transaction_id":"1000000157783770",
            "bvrs":"2.6.0",
            "transaction_id":"1000000157783770",
            "quantity":"1",
            "unique_vendor_identifier":"SECRETCD-89AD-45C4-8937-359CCA9E8F36",
            "item_id":"SECRET509",
            "product_id":"com.your.iap.product.id",
            "purchase_date":"2015-06-03 11:00:37 Etc/GMT",
            "original_purchase_date":"2015-06-03 11:00:37 Etc/GMT",
            "purchase_date_pst":"2015-06-03 04:00:37 America/Los_Angeles",
            "bid":"com.your.app.bundle.id",
            "original_purchase_date_ms":"1433329237329"
            },
"status": 0
}
```
第5点中提到过，后台需要在发起支付前针对每一笔支付生成一个订单。服务端使用客户端发来的receiptData得到苹果的二次验证返回时，首先比较订单中的商品和返回值中的 product_id 是否对应，若不一致则用户用作弊手段传了另一个商品的 receiptData 过来，视本次支付无效。若一致，则需要判断返回值中的unique_identifier是否被使用过；若未被使用过，则视此次交易完成，并将此unique_identifier标记为已使用；若使用过，则用户使用作弊手段传了之前购买时的 receiptData 过来。  

## 7.切换线上/测试环境

需要在代码里显示声明的环境，就只有二次验证地址：  

```objective-c
#define SANDBOX_VERIFY_RECEIPT_URL          [NSURL URLWithString:@"https://sandbox.itunes.apple.com/verifyReceipt"]
#define APP_STORE_VERIFY_RECEIPT_URL        [NSURL URLWithString:@"https://buy.itunes.apple.com/verifyReceipt"]
```

而调用苹果接口时连接的是线上环境还是测试环境，猜测是由编译 App 的证书决定的。目前看来，开发证书编译后，连接的是苹果的 Sandbox 环境；AppStore 上下载的则是连接苹果的线上环境。  

另外再次强调，除非少量必要的自己线上环境的测试需要连接苹果的 Sandbox 验证服务之外，自己服务端的二次验证 API 应该严格做到自己的环境是线上环境，则连接苹果的线上环境二次验证接口。防止监守自盗的情况出现。  

Over

