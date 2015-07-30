---
layout: post
title: "In-App Purchase Walk Through"
date: 2015-02-28 16:19:15 +0800
comments: true
categories: ["iOS", "IAP"]
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

1. 现实世界的商品或服务：比如刚才提到的一次「性服务」。严格遵守此方案有个好处：IAP 如果被破解，用户无法得到大量实物，开发商也不会有很大经济损失。非要做的话想绕过也是可以的：用 IAP 购买代币，审核通过后用代币购买实物。  
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
在收到Purchased或Restored回调后，持久化购买记录以及receipt data。iOS6.X 或之前的版本中，持久化 receipt data 必须万无一失，因为一旦丢失，将没有任何途径再次拿到此 receipt，造成用户购买记录丢失。获取 receipt data 需要注意的点将在后面的二次验证中详细说。  
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
    NSData *transactionReceipt = [[self class]] receiptDataFromTransaction:transaction];
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

+ (NSData *)receiptDataFromTransaction:(SKPaymentTransaction *)transaction
{
    NSData *receiptData = [self receiptDataInReceiptURL];
    if (!receiptData)
    {
#pragma clang diagnostic push
#pragma clang diagnostic ignored "-Wdeprecated"
        if ([transaction respondsToSelector:@selector(transactionReceipt)])
        {
            //Works in iOS3 - iOS8, deprected since iOS7, actual deprecated (returns nil) since iOS9
            receiptData = transaction.transactionReceipt;
        }
#pragma clang diagnostic pop
    }
    return receiptData;
}

+ (NSData *)receiptDataInReceiptURL
{
    if (SYSTEM_VERSION_GREATER_THAN_OR_EQUAL_TO(@"7.0") && [[NSBundle mainBundle] respondsToSelector:@selector(appStoreReceiptURL)])
    {
        //Works since iOS7, implemented but calls selector not found directly in iOS6
        //so must decide by if system version >= 7.0, DO NOT use respondsToSelector:@selector(appStoreReceiptURL)
        NSURL *receiptUrl = [[NSBundle mainBundle] appStoreReceiptURL];
        if ([[NSFileManager defaultManager] fileExistsAtPath:[receiptUrl path]])
        {
            NSData *receiptData = [NSData dataWithContentsOfURL:receiptUrl];
            return receiptData;
        }
    }
    return nil;
}

```

需要注意的是，如果 App 不需要支持 iOS6.x 及之前的版本，建议仅使用 appStoreReceiptURL 获取 receipt 数据。appStoreReceiptURL 从 iOS7 开始启用，会返回用户在此 app 上购买过的全部 receipt 的 data（粒度猜测应该是本机，本 App Store 帐号，本 App 内的购买，具体没测试）。  

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

使用transaction.transactionReceipt取得的小票经二次验证后返回值：  

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

使用[[NSBundle mainBundle] appStoreReceiptURL]取得的小票经二次验证后返回值：  

```JSON
{
"status":0,
"environment":"Sandbox",
"receipt":  {
            //in_app 中全部支付共有的信息（本App的IAP信息）
            "receipt_type":"ProductionSandbox",
            "adam_id":0,
            "app_item_id":0,
            "bundle_id":"com.netease.neteasemusic",
            "application_version":"2.9.0",
            "download_id":0,
            "version_external_identifier":0,
            "request_date":"2015-07-29 03:37:17 Etc/GMT",
            "request_date_ms":"1438141037628",
            "request_date_pst":"2015-07-28 20:37:17 America/Los_Angeles",
            "original_purchase_date":"2013-08-01 07:00:00 Etc/GMT",
            "original_purchase_date_ms":"1375340400000",
            "original_purchase_date_pst":"2013-08-01 00:00:00 America/Los_Angeles",
            "original_application_version":"1.0",
            "in_app":   [
                            //每笔交易分别的信息
                            {
                                "quantity":"1",
                                "product_id":"com.netease.neteasemusictest.pack.clouddrive.2t12",
                                "transaction_id":"1000000164784521",
                                "original_transaction_id":"1000000164784521",
                                "purchase_date":"2015-07-23 15:03:04 Etc/GMT",
                                "purchase_date_ms":"1437663784000",
                                "purchase_date_pst":"2015-07-23 08:03:04 America/Los_Angeles",
                                "original_purchase_date":"2015-07-23 15:03:04 Etc/GMT",
                                "original_purchase_date_ms":"1437663784000",
                                "original_purchase_date_pst":"2015-07-23 08:03:04 America/Los_Angeles",
                                "is_trial_period":"false"
                            },
                            {
                                "quantity":"1",
                                "product_id":"com.netease.neteasemusictest.pack.clouddrive.1t12",
                                "transaction_id":"1000000165136224",
                                "original_transaction_id":"1000000165136224",
                                "purchase_date":"2015-07-27 06:40:40 Etc/GMT",
                                "purchase_date_ms":"1437979240000",
                                "purchase_date_pst":"2015-07-26 23:40:40 America/Los_Angeles",
                                "original_purchase_date":"2015-07-27 06:40:40 Etc/GMT",
                                "original_purchase_date_ms":"1437979240000",
                                "original_purchase_date_pst":"2015-07-26 23:40:40 America/Los_Angeles",
                                "is_trial_period":"false"
                            }
                        ]
            }
}
```

#### 纯本地验证
除了网络验证以外，苹果提供了纯粹的本地验证方式：[Validating Receipts Locally](https://developer.apple.com/library/ios/releasenotes/General/ValidateAppStoreReceipt/Chapters/ValidateLocally.html#//apple_ref/doc/uid/TP40010573-CH1-SW2).  
Receipt data 经过 App Store 证书签名，所以第三方无法凭空生成能够通过此法验证的 receipt data。只要做好证书校验，无需担心用户会伪造 receipt data。  
在客户端使用这种方式可以做到防止被通用破解方式破解，但并不能防止针对特定 App 的破解。  
实际上，这种验证方式是苹果为服务端设计的。Receipt data 的格式遵守ASN.1格式，服务端安装asn1c就可以解析 receipt data，并不需要纯手写一份解析代码。只要服务端代码和 asn1c 不出 bug，在服务端使用这种方式验证就是安全的。  

#### 第三方网站验证
有些第三方网站提供了经服务端的验证服务。比如[urbanairship](http://urbanairship.com/products). 但是我并没有用过，所以不知道具体效果如何。毕竟第三方服务无法做到在用户发起购买之前生成订单记录，与购买后验证结果比对，所以我还是比较担心第三方验证服务的安全性的。而且鸡国网络连国外验证服务器，你懂的。。

总之想要万无一失，建议开发自己的验证接口。  

更多验证相关问题，请参考[Receipt Validation Programming Guide](https://developer.apple.com/library/ios/releasenotes/General/ValidateAppStoreReceipt/Introduction.html#//apple_ref/doc/uid/TP40010573)

大多数产品在验证成功后，才是真正的发放内容、道具等。特别是充值后立即消费的虚拟货币基本都是这么处理的。  
但是从接口来看， IAP 的设计者是想让开发者在购买完成时发放内容、道具，在二次验证失败时以删除内容、道具等方式来进行处罚。  

## 6.服务端二次验证后再发放数据中的安全问题

由于是和钱关系最紧密的功能，IAP安全性显得无比重要。  

### 客户端数据安全

客户端根据使用不同的获取 receipt 的接口，需要做的事情也不同：

####1. 客户端使用transaction.transactionReceipt（或使用transaction.transactionReceipt + appStoreReceiptURL两者）:  

如果要支持 iOS6，那么不得不使用transaction.transactionReceipt在 iOS6上读取receipt，客户端需要保证持久化逻辑：  
在transaction完成后，和服务端的二次验证完成前，要对receipt data做持久化；
若存在未上传成功的 receipt ，需要开定时器重试上传；
如果要做删除持久化数据，删除的时机应当是收到从服务端发回的二次验证请求的响应时，确认服务端已和苹果完成通信之后（服务端返回和苹果连接失败则不应删除已保存的receiptData），若 IAP 交易不频繁，可考虑不删除持久化的 receipt data。  

####2. 客户端仅使用appStoreReceiptURL获取receipt  

客户端每次交易完成从 appStoreReceiptURL 读取出 data 上传给服务器。同时有上传请求正在进行的话，持久化一个标记，表示有未完成的上传，在全部上传完成后将此标记置为上传结束。如果标记存在，需要开定时器重试上传。

### 服务端数据安全  

服务端安全包含两部分：防止已扣款却购买无效；防止作弊。目前想到最圆的实践是：  
1. 在从客户端上传的 receipt data 中解析出的数据（包括本地解析或经苹果服务器解析）中，从全部交易记录中找到所有未被标记为已使用的交易记录，作为集合A。  
2. 枚举当前用户的全部待支付订单，匹配集合A中的交易记录中product id相同的项，并将此交易记录的 transaction id 记录下，标记为已使用；并将此订单标记为已支付，并发放道具。  
3. 线上环境仅在审核期间允许使用sandbox环境做二次验证，防止上线后内部同事使用sandbox test user免费充值道具到线上环境。需注意审核期间必须开启 sandbox 环境验证，不然会被 rejected。  

### 其他安全问题

1. 尽量使用纯 HTTPS 接口上传 receipt，并严格校验 SSL 证书，防止中间人攻击。  
2. 越狱后有木马会盗取用户的支付，如果必要，需要提醒用户越狱风险。  

## 7.切换线上/测试环境

需要在代码里显示声明的环境，就只有二次验证地址：  

```objective-c
#define SANDBOX_VERIFY_RECEIPT_URL          [NSURL URLWithString:@"https://sandbox.itunes.apple.com/verifyReceipt"]
#define APP_STORE_VERIFY_RECEIPT_URL        [NSURL URLWithString:@"https://buy.itunes.apple.com/verifyReceipt"]
```

而调用苹果接口时连接的是线上环境还是测试环境，猜测是由编译 App 的证书决定的。目前看来，开发证书编译后，连接的是苹果的 Sandbox 环境；AppStore 上下载的则是连接苹果的线上环境。  

另外再次强调，除非少量必要的自己线上环境的测试需要连接苹果的 Sandbox 验证服务之外，自己服务端的二次验证 API 应该严格做到自己的环境是线上环境，则连接苹果的线上环境二次验证接口。防止监守自盗的情况出现。  

## 8.提交审核

如果是初次提交审核，IAP 商品要和第一个支持 IAP 的版本一起提交。审核期间要允许 sandbox 环境二次验证。  
后续新增的 IAP 商品则没有此限制，可以随时提交审核。  

Over

