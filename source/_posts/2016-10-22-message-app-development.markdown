---
layout: post
title: "Message app开发文档（Stickers与Messages Extension）"
date: 2016-10-22 04:22:11 +0800
comments: true
categories: [iOS, Messages extension, Message app]
---

## Messages App 概述

开发者可以为 iOS 10 创建 Messages app extension。用户在其中可以发送文字、表情、媒体文件、交互式消息（交互式消息即 interactive messages，是一种允许被对话的参与者更新状态的消息）。  

第三方软件可以使用 Messages framework 创建两种 app extension: 表情包（Sticker packs）和 Message apps。这两种 app extension 都可以作为现有主 app 的 extension 来发布，也可以单独发布。  

**为了方便理解后文内容，强烈建议先玩一下：**  
1. Message表情比如知乎刘看山  
2. GamePigeon中的台球  
3. 大众点评的订电影票  

<!--more-->

## 表情包简介

表情包（Sticker packs）是一组不能变更（指不允许增删改图片文件，允许gif动图）的表情图片。表情图片可以被当做一条消息单独发送，也可以附加到一条已有消息的气泡上。创建表情包并不需要写任何代码，把图片拖拽到 Stickers asset catalog 中的 Sticker Pack 文件夹下即可。用作表情的图片必须符合以下规范：  

* 必须是 PNG, APNG, GIF, 或 JPEG 格式的文件。  
* 每张图片必须小于 500 KB。  
* 图片不能小于 100 x 100 points 或大于 206 x 206 points，以保证最佳显示效果。  
* 仅提供 @3x 的图片即可（在 300 x 300 像素到 618 x 618 之间）。系统会在运行时缩放生成 @2x 和 @1x 的版本。  

Messages 仅支持三种尺寸的表情。表情被显示在 grid-based browser中。表情图片尺寸需要在 Xcode 的 Attributes inspector 中设置。支持的尺寸有：  

* 小图： 100 x 100 points @3x (300 x 300 pixels).  
* 中图： 136 x 136 points @3x (408 x 408 pixels).  
* 大图： 206 x 206 points @3x (618 x 618 pixels).  

####适用范围：
1. 除了纯粹做表情  
2. 可以为 app 做 branding  
3. 除此之外似乎并没什么卵用  

更多关于创建 sticker packs 的话题，参考[Creating Stickers with Motion](https://developer.apple.com/support/stickers/motion/)。  

## Message Apps 简介

Message apps 可以用来：  

* 在Messages app呈现自定义UI：参考 [MSMessagesAppViewController](https://developer.apple.com/reference/messages/msmessagesappviewcontroller?language=objc)。

* 创建自定义或动态 sticker browser：参考 [MSStickerBrowserViewController](https://developer.apple.com/reference/messages/msstickerbrowserviewcontroller?language=objc)。

* 向 Messages app 的输入框插入文字、表情或媒体文件：参考 [MSConversation](https://developer.apple.com/reference/messages/msconversation?language=objc)。  

* 创建携带 app 特定数据的交互式消息：参考 [MSMessage](https://developer.apple.com/reference/messages/msmessage?language=objc)。

* 更新交互式消息 (比如游戏或合作类 apps----举个例子，台球或大众点评约朋友一起电影)：参考 [MSSession](https://developer.apple.com/reference/messages/mssession?language=objc)。

#### 适用范围
当用户点击一条交互式消息时，系统会弹出一个 controller ，开发者可以在其中做很多事。  
Messages extension 可以与主 app 共享数据，也能单独做一个完整应用，也有单独访问网络权限。    
适合交互式场景的应用或游戏。  
适合需要分享和 IM ，但是客户端中内嵌 IM 有点重的 app。

更多创建 app extension 的话题请参考 [App Extension Programming Guide](https://developer.apple.com/library/content/documentation/General/Conceptual/ExtensibilityPG/index.html#//apple_ref/doc/uid/TP40014214)。

## Symbols

#### Classes

1. [MSConversation](https://developer.apple.com/reference/messages/msconversation?language=objc)  
MSConversation 在 Messages app 中用来表示一组对话。使用 conversation 对象可以获取当前单聊或者群聊中的信息，或者发送文字、表情、附件或其他消息对象。  

2. [MSMessage](https://developer.apple.com/reference/messages/msmessage?language=objc)  
MSMessage 用来创建交互式消息对象。如果要创建一个能被对话中的其他人更新状态的消息，使用`initWithSession:`初始化，否则使用`init`初始化。  

3. [MSMessageLayout](https://developer.apple.com/reference/messages/msmessagelayout?language=objc)  
MSMessageLayout 是用来定义 MSMessage 显示样式的抽象基类，每种子类对应一个显示样式，然而，目前只有一个子类`MSMessageTemplateLayout`。  

4. [MSMessagesAppViewController](https://developer.apple.com/reference/messages/msmessagesappviewcontroller?language=objc)  
MSMessagesAppViewController 是 Messages extension 中的 principal view controller，当 Message app 换起 extension 时，这个controller 就会出现。向其添加不同的子 controller 以实现不同的 feature。    

5. [MSMessageTemplateLayout](https://developer.apple.com/reference/messages/msmessagetemplatelayout?language=objc)  
MSMessageTemplateLayout 是 MSMessageLayout 的子类。描述 MSMessage 在消息中如何显示。消息模板包含 message extension 的 icon，一份图片/视频/音频文件，以及一组文字元素（title, subtitle, caption, subcaption, trailing caption, and trailing subcaption）。  

6. [MSSession](https://developer.apple.com/reference/messages/mssession?language=objc)  
用来创建交互式消息的对象。    

7. [MSSticker](https://developer.apple.com/reference/messages/mssticker?language=objc)  
Messages app 中的表情对象。Sticker 可以被单独发送，也可以被附加到一条已存在的消息的气泡上。   

8. [MSStickerBrowserView](https://developer.apple.com/reference/messages/msstickerbrowserview?language=objc)  
MSStickerBrowserView 用来显示动态生成的表情列表。用户用户可以点击 browser 中的表情将其加到输入框内，也可以将表情从 browser 中拖拽到任意消息气泡上。  

9. [MSStickerBrowserViewController](https://developer.apple.com/reference/messages/msstickerbrowserviewcontroller?language=objc)
MSStickerBrowserViewController 用来显示标准表情 browser。和MSStickerBrowserView一样，用户点击表情插入输入框内，或拖拽表情来附加到任意气泡上。

10. [MSStickerView](https://developer.apple.com/reference/messages/msstickerview?language=objc)  
MSStickerView 用来显示表情对象。和上面一样，点击插入输入框，拖拽附加到气泡。  

#### Protocols

1. [MSStickerBrowserViewDataSource](https://developer.apple.com/reference/messages/msstickerbrowserviewdatasource?language=objc)  
动态为表情 browser 提供表情时需实现此协议。

#### 相关文档

1. [Errors](https://developer.apple.com/reference/messages/2216109-errors?language=objc)  
此文档描述了表情包和 Message apps 中使用的错误类型和错误代码（error code not error source code）。  
2. [Messages Enumerations](https://developer.apple.com/reference/messages/1990031-messages_enumerations?language=objc)  
MSMessagesAppPresentationStyle:  描述 controller 的展示样式，是收起在底部，还是伸展到全屏。  
MSStickerSize :  描述表情的显示尺寸。  
