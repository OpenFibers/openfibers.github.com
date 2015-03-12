---
layout: post
title: "Apple Watch人机交互指南"
date: 2015-03-11 13:49:29 +0800
comments: true
categories: ["WatchKit", "iOS", "Apple Watch"]
---

翻译自[官方文档 - Apple Watch Human Interface Guidelines](https://developer.apple.com/library/prerelease/ios/documentation/UserExperience/Conceptual/WatchHumanInterfaceGuidelines/index.html)。

文档主要分为三部分：  
1. 为Apple Watch设计软件的注意事项。包括介绍了watch软件的特点、watch独有的新的交互方式、Glances、Notifications、Modal Sheets、布局、颜色与排版、动画、app品牌化，也穿插了一些设计原则；  
2. 各种控件的使用与设计原则。包括labels、images、groups、tables、buttons、switches、sliders、maps、dates、times和menus。虽然控件种类不多，但是和以往相比区别还是比较大的；  
3. Icon、image和menu image的设计原则。  

苹果的人机交互文档不只是给码农看的，也是给PD与设计师看的。建议PD和设计师们也读一读。  

# 为Apple Watch设计软件的注意事项：

## Apple从未有过的穿戴式设备
1. Personal: 一方面更隐私，另一方面更个人化。带有心率等其它传感器，可以得到比以往设备更多的个人信息。由于是苹果的首个穿戴式设备，从未有过一款设备像apple watch一样如此与使用者紧密相连，这一点在设计应用时要多加注意。  
2. Holistic: Apple watch是软件与硬件的结合。新的交互方式包括Digtal Crown(表冠)、Taptic Engine(触觉反馈+细微声音反馈)和Force Touch(压力感应触摸屏)，[官方相关介绍传送门](https://www.apple.com/cn/watch/technology/)。设计App时应当把以上交互体验一并考虑在内。    
3. Lightweight: Apple watch上的软件定义为轻量级的，要求能快速启动，交互简洁、UI简单。  

## UI架构与交互方式

### UI类型
App有两种基本样式，基于继承的、基于平行页面的。  

1. Hierarchical：更像iOS上的导航模式。有推入推出。适合处理复杂的业务逻辑。  

{% img left /images/blog/apple_watch_guidelines/1.hierarchical_interface_2x.png 513 267 %}  

2. Page-based：几个页面横滑切换，而且苹果希望尽可能少的并列页面。适合处理简单的业务逻辑。  

{% img left /images/blog/apple_watch_guidelines/2.paged_interface_2x.png 679 267 %}  

这两种基本架构不允许同时存在，只能二选其一。  

### 用户交互

1. Action-based：单击操作。在各种各样的控件上点的事件都是。  
2. Gestures：竖滑、横滑、左边缘右滑后退、点击。  
3. Force Touch：大力按下时系统会弹出context menu，作用类似pc上的右键菜单。App内当使用此menu显示上下文可以操作的actions。  
4. The Digital Crown：第三方应用仅可以使用表冠卷动来支持滚动长页面。  

<!--more-->

## Glances

每个App可以创建自己的Glance，Glance应当是自己应用中最关键内容的展示。  
用户从安装的App中选出一些喜欢应用的Glance，这就组成了Glances。  
有点像iOS的Extension。  

{% img left /images/blog/apple_watch_guidelines/3.glances_2x.png 486 267 %}  

Glances的特点:  
1. Template-based: 使用Xcode选取合适的模板，然后根据模板设计合适的内容。  
2. Not scrollable: 不可滚动。只能显示一屏。  
3. Associated with a single action: 点击Glance的任意位置都是相同的action，此时跳入app后，然后进入恰当的页面。  
4. Optional: 不是所有的app都需要做这个。可以不做。  

## Notifications

Notifications是在apple watch上用于处理远程通知或本地通知的方便轻量的交互。  
无论是远程通知，还是本地通知，都分为两个阶段。第一个阶段：Short Look, 在通知到达的第一时间展示给用户，应谨慎选择内容，给用户提供最少量的信息，以保护用户隐私。如果用户低下手腕，Short Look会自动隐藏。如果用户点击Short Look或者抬起手腕，Look Look将会显示出来。Long Look应当显示通知的详细信息，而且Long Look只能被用户关闭。  

### Short Look Notifications

Short Look用于告知用户收取了通知，并展示简略信息。和Glance一样，是template-based，里面包含app图标、app名称和通知标题。系统使用app的key color显示app名称：  
{% img left /images/blog/apple_watch_guidelines/4.shortlook_calendar_2x.png 488 337 %}  
通知标题可用的空间是相当小的，所以标题应该尽量简短，用于向用户简要描述此通知是做什么的。  

### Custom Long Look Notifications

Long Looks提供通知的详细信息。App可以自行定制Long Look的视图，也可以使用系统提供的默认视图。所有App的Long Look结构都是一样的。系统将会把app icon和app名称显示在顶部的sash area，sash area会给其下层的内容增加模糊效果。在通知的最底部是action按钮，app可以增加自定义action按钮，系统将会添加Dismiss按钮。在sash area和底部action按钮之间的中间部分，用于展示通知内容。  

{% img left /images/blog/apple_watch_guidelines/5.longlook_calendar_2x.png 479 703 %}  

关于Long Look Notifications：  

1. 通知内容可以垫在sash area的下层（underlap），透过半透明blur效果显示出来，也可以紧贴着显示在sash area的下面(just below)。一般来说图片可以选择垫在下层，而文字要紧贴着显示在sash area下面。  
{% img left /images/blog/apple_watch_guidelines/6.notification_long_content_2x.png 471 457 %}  
App需要定制Long Look显示的话, 必须提供static interface，也可一并提供dynamic interface。Static interface和dynamic interface都显示相同的通知类型。Dynamic interface比static interface定制性更强，而当dynamic interface不可用时，static interface提供了备用方案（fallback position）。关于static interface和dynamic interface在[Apple Watch Programming Guide](https://developer.apple.com/library/prerelease/ios/documentation/General/Conceptual/WatchKitProgrammingGuide/index.html#//apple_ref/doc/uid/TP40014969)中有详细介绍。  
2. Dismiss按钮将会始终显示。  
3. 除dismiss按钮之外，还可另外显示最多四个自定义action按钮。首先App向系统注册交互通知，然后Apple Watch根据注册的交互通知的类型来显示不同的action按钮。如何显示action自定义按钮在[Apple Watch Programming Guide](https://developer.apple.com/library/prerelease/ios/documentation/General/Conceptual/WatchKitProgrammingGuide/index.html#//apple_ref/doc/uid/TP40014969)中有详细介绍。  
4. Sash area的颜色和透明度是可以定制的。  

关于static and dynamic interfaces, 以及如何设置action按钮，移步[Apple Watch Programming Guide](https://developer.apple.com/library/prerelease/ios/documentation/General/Conceptual/WatchKitProgrammingGuide/index.html#//apple_ref/doc/uid/TP40014969)。

## Modal Sheets

{% img left /images/blog/apple_watch_guidelines/7.modal_interfaces_2x.jpg 547 343 %}  

Model sheet是一种弹出的视图，可以包含一屏子视图，也可以使用page-based布局包含多屏子视图。弹出时会临时阻挡软件其他部分的交互。  
一般用于：  

1. 让用户完成某个task；  
2. 向用户展示信息并引起用户的注意；  
3. 让用户继续选择子选项。  

App中应当尽量减少modal sheet的使用次数。一般来说，仅在满足下列条件时（二者皆满足或二者之一被满足）使用：  

1. 弹出的内容对于获取用户的注意是至关重要的；  
2. 包含的task必须被完成或隐式被禁用，以便避免用户数据处于模糊状态。  

注意事项：  

1. 左上角的关闭按钮是不能隐藏的，但是可以更改文案。但功能还是关闭作用。  
2. 如果需要提供给用户'接受'操作，请在modal sheet试图body上增加一个标准按钮。  
3. Modal task应该尽量简单，且避免从一个modal sheet进入另一个modal sheet。

## Layout

### Layout指南

1. 减少并列排放的控件数量。当按钮需要并列排放时，应当使用图标代替文字贴在按钮上。永远不要让三个以上的控件并列排放，不然按钮太小，用户很难点击到。  
2. 所有控件应该贴紧屏幕左右边缘显示，在物理机上，屏幕已在app内容外留有间距。注意这一点在模拟器上是看不到的。  
{% img left /images/blog/apple_watch_guidelines/8.fullwidth_settings_2x.png 272 700 %}  
3. 为控件使用相对定位，因为手表屏幕有不同的尺寸。  
4. 控件尽量使用从上到下、从左到右的对齐方式。  
5. 文字按钮宽度应该使用最长宽度。这样按钮上的文字才是始终可见的。  
6. 对于使用偏少的操作，使用上下文菜单来代替增加按钮，毕竟屏幕太小。  
{% img left /images/blog/apple_watch_guidelines/9.menu_stopwatch_2x.png 272 387 %}  

### 屏幕尺寸

1. 不管大屏幕还是小屏幕，应当使用相同的UI，然后自然拉伸或扩展。  
{% img left /images/blog/apple_watch_guidelines/10.watch_screen_sizes_2x.png 637 454 %}  
2. 如果一张资源图可以兼容两种屏幕尺寸，那么准备一张就可以了，比如小的按钮。如果无法兼容两种屏幕尺寸，那么准备两张，比如填满屏幕的表盘背景。  

## Color and Typography

### 颜色

主题颜色让每个App显得与众不同。颜色也为App提供了视觉连续性。  

{% img left /images/blog/apple_watch_guidelines/11.colors_calendar_list_2x.jpg 272 666 %}  

注意事项：  

1. 一定要使用黑色作为App背景色。这样可以与表框无缝融合，造成一种没有屏幕边缘的视觉错觉。其他控件也要避免使用亮色作为背景。  
2. 定义一个key color，Key color会用于显示软件信息，例如屏幕左上角的标题文字，以及Notification中的软件名和其他关键信息。Key color也应当被用在其他控件中，用来brand app，就像阿里橙和微信绿那样。  
3. 为文字使用高对比度的字色，这样在黑色背景中才会清晰。  
4. 为视觉障碍患者考虑切身感受，比如绿色的取消和红色的删除按钮并列显示时，如果两个字色亮度一样，红绿色盲将不能通过颜色得知两个按钮的区别。两个按钮字色使用不同亮度将使这一情况得到改善。  
5. 颜色是一种语言，但并不一定能准确表达你的意思。同种颜色在不同的人眼里是不一样的，而且在不同的国家、不同的文化中，同一个颜色代表的意义也可能不同。你应该尽可能多的保证使用的颜色表达了你想表达的意思。  

### 文字排版

首先，文字无论如何要保证清晰。如果用户看不清你的文字，再清晰的排班都没用。  

{% img left /images/blog/apple_watch_guidelines/12.typography_mail_2x.png 272 337 %}  

苹果本着在手表能够清晰显示的原则设计了手表的系统字体。在字体尺寸稍大时，字母与字母之间会稍微减少间距，以便每横排能展示更多的内容。字体尺寸稍小时，字母间距会增加，并且像字母a和字母e的孔径会增大，以便让用户一眼就能看清，同时标点也会适当按比例变大。为使文字一直保持清晰易阅读的状态，当字体尺寸改变时，系统将会动态做出上述优化。  

Dynamic Type是一种使字体变的更清晰易读的技术。App应该始终使用Dynamic Type技术。当使用Dynamic Type技术时，会有以下几点相当便利：

1. 行高和字母间距将为每一种字体尺寸自动优化。  
2. 可以为不同的文字区块（例如Body, Footnote, 或者Headline）指定不同的text style。  
3. App内的文字能够正常响应用户在系统设置里做的文字大小设置，包括accessibility中的文字大小设置。  

如果使用系统内建的text style，Dynamic Type将会自动生效，无需额外做任何工作。如果使用自定义字体，软件内必须做一些事情来支持这个feature。关于如何使用text styles，以及软件如何在用户更改系统字体大小设置时获得通知，移步[Text Styles in Text Programming](https://developer.apple.com/library/prerelease/ios/documentation/StringsTextFonts/Conceptual/TextAndWebiPhoneOS/Introduction/Introduction.html#//apple_ref/doc/uid/TP40009542)中的[Text Styles](https://developer.apple.com/library/prerelease/ios/documentation/StringsTextFonts/Conceptual/TextAndWebiPhoneOS/CustomTextProcessing/CustomTextProcessing.html#//apple_ref/doc/uid/TP40009542-CH4-SW65)章节。  

注意事项：  
1. 因为天然支持Dynamic Type，内建的text style看起来更好些，能用内建的text style就尽量用。  
{% img left /images/blog/apple_watch_guidelines/13.text_styles_2x.png 312 387 %}  
2. 整个app最好只使用一种字体。混合多种字体让app看起来更松散和零碎。  
3. When specifying system font sizes manually, the point size determines the correct size to use. Choose the San Francisco Text font for text that is 19 points or smaller. Choose the San Francisco Display font for text that is 20 points or larger.(这句话不知道如何理解)  

## Animations

Apple Watch中有很多优雅微妙的动画，让用户体验变得更好、更活跃。  
适当的动画效果可以反馈app中的一些状态，或是视觉化用户正在进行的操作。  

注意事项：

1. 请使用预渲染的静态图片组成连续的动画。将连续的静态帧储存在app bundle中，以便快速向用户展示。预渲染的静态帧能够提升帧率，让动画更流畅。在WatchKit中动态创建动画（先使用CoreGraphics逐帧生成静态图），相比使用预渲染的静态帧来说会稍有延迟。  
2. 程序可控的回放控制仅仅在image对象和group对象上可用。大部分控件无限循环的播放动画，如果想暂停动画，或者使用一组选定的静态帧播放动画，必须使用[WKInterfaceImage](https://developer.apple.com/library/prerelease/ios/documentation/WatchKit/Reference/WKInterfaceImage_class/index.html#//apple_ref/occ/cl/WKInterfaceImage)或[WKInterfaceGroup](https://developer.apple.com/library/prerelease/ios/documentation/WatchKit/Reference/WKInterfaceGroup_class/index.html#//apple_ref/occ/cl/WKInterfaceGroup)中的方法。  

## Branding
品牌化app有许许多多的方式，包括图标、颜色、自定义按钮、自定义字体、文案等。设计app的视觉元素时，每个元素都应拥有漂亮的外观和自己良好的功能性，也应该与其他元素（自定义元素或标准元素）看上去像是同属一个app的整体。  

注意事项：  
1. 用优雅、不显眼的方式合并品牌化的元素，不要大肆渲染品牌。用户使用app的目的是完成某件事或者娱乐，他们并不想产生看广告的感觉。为了提供最好的用户体验，应当使用一些较软性、易接受的方式产生品牌的存在感，无论通过字体、颜色或是图片。  
2. 在app内或Glance中避免使用软件logo。Apple Watch的屏幕很小，logo将会占用用户想看的内容的空间。除此之外，在app中显示logo并不能达到在web页中显示logo相同的目的：web页可以通过链接点入，用户点入前并不知道链接属于谁，而app都是从icon点进来的。  

# 控件介绍

## Labels
用于显示静态文字。  
{% img left /images/blog/apple_watch_guidelines/14.typography_mail_2x.png 272 337 %}  

特点：  
1. Label可以显示一定数量的静态文字  
2. 不接受用户点击等交互操作  
3. 可以使用程序更改内容  
4. 可以折行  
5. Label是最常用的控件之一，用于向用户显示简短的信息  
6. 最适用于显示相对较少的文字  

注意事项：  

1. 让用户能看清label。使用高对比度的颜色和Dynamic  Type技术来保证良好的显示效果。推荐使用内建的系统字体，因为内建系统字体在Apple Watch上是最清晰的。如果你想使用自定义字体，不要为了花哨的字母效果、华丽的颜色而牺牲清晰度。  
2. 能用内建text style就用内建的。内建的text style是最清晰的。  

关于更多在app内显示文字的话题，包括如何使用Dynamic type，移步[Color and Typography](https://developer.apple.com/library/prerelease/ios/documentation/UserExperience/Conceptual/WatchHumanInterfaceGuidelines/ColorandTypography.html#//apple_ref/doc/uid/TP40014992-CH9-SW1).

## Images

Image对象用来显示一张静态图或一组动态图。  
{% img left /images/blog/apple_watch_guidelines/15.images_worldclock_2x.jpg 272 337 %}  
特点：  
1. Image对象并没有加在上面的任何额外元素或特效，它仅仅显示它自己的图片  
2. 不接收点击等交互操作  
3. 提供代码层面的接口来控制动画的开始和暂停  

注意事项：  
1. 不要使用同一份图片文件在不同屏幕尺寸上拉伸或压缩，不然图片会变模糊。当图片需要在不同尺寸的屏幕上以不同尺寸显示时，应为图片添加图片组，里面放置不同尺寸的图片。  
2. 所有图片都准备@2x的图片就可以了。没必要准备非视网膜屏幕的图片，以后也大概不会有非视网膜屏的Apple Watch，目前也没有3x视网膜屏的Apple Watch。  

## Groups

Groups是布局的重要工具。Group是其他控件的容器。Group自己除了可以设置背景颜色或背景图片外，没用任何自己的样式。Group有position、size、margin和其他布局相关的属性。  

特点：  
1. 内部可以横向或纵向排布其他控件  
2. 可以包含一个或多个控件元素  
3. 可以为内部元素定义margin和spacing属性  
4. 可以将一张图片或颜色作为背景显示  
5. 可以为背景和内容设置圆角半径，让它变成一个圆角矩形  

Group是布局的首选控件。尽管如此，你可以把他单独作为一个控件使用，毕竟能设置背景与圆角。  

注意事项：  

1. Group可以内嵌一个或多个group来构成复杂的视图。如此做变可以让一些元素横向排布、另一些元素纵向排布。Margin或spacing选项也是一个很好的利用点。  
2. 如果要使用图片做背景，和Image对象的要求一样，图片不要压缩或拉伸，不然图片会变模糊。当图片需要在不同尺寸的屏幕上以不同尺寸显示时，应为图片添加图片组，里面放置不同尺寸的图片。  

## Tables

和iPhone上的UITableView一样，Table内含多行单列。Table可以用来展示动态变更的内容。  
{% img left /images/blog/apple_watch_guidelines/16.table_list_2x.png 272 337 %}  

特点:  
1. 可以拥有多行  
2. 支持滚动  
3. 可以设置背景图片或背景色  

必须在设计期（编译期）指定table支持的全部的row types，在运行期，你可以选择具体使用哪种row type。  

注意事项：  
1. 整个app中都要使用row types。你可以为header、footer或content创建不同的row type。然后需要注意，不要为header使用content的row type，反之亦然。  
2. 避免把很多row types混在一起用。显示某种内容应当使用统一的row type。只有在做section分割或用来组织不同的内容时，才应当使用不同的row types。同种内容统一的row type可以保证这些行可以以统一的方式布局，而且对于用户来说也易于导航。  
3. 同个table中避免显示过多行。超过20行的table滚动起来是十分痛苦的，不然一年内拧断了表冠还得免费给用户换新。Table中只从行中选出一些直接相关的来显示，然后给用户加载更多行的选项。  
4. 不要把table放在group中。Table根据行数的不同动态变更尺寸，于是table会忽视group中设置的高度限制选项。  

## 按钮
按钮可以发起特定的动作。  
{% img left /images/blog/apple_watch_guidelines/17.buttons_calendar_2x.png 272 337 %}  

特点：  
1. 背景可以自定义  
2. 四个角是圆角的，不该不带圆角  
3. 可以内嵌label或group对象  

Button的背景像个盘子一样。背景色或背景图可以在运行期被修改。  

注意事项：  
1. 按钮最好横向填满屏幕。如果必须横向排布按钮的话，最多排布两个。  
2. 同一屏内容上的按钮高度应当保持一致。  
3. Button使用圆角和其他元素相区分。标准圆角半径是6px。  

## Switches
Switch让用户选取互斥的状态。  
{% img left /images/blog/apple_watch_guidelines/18.switches_2x.png 486 267 %}  

特点：  
1. 表明某种信息的二选一状态  
2. 永远都会自带一个label  

使用switch来让用户从两个选项之中选取一个。  

## Sliders

Slider可以让用户从一个范围内选取数值。和iPhone上的slider不同，这个slider调整数值是靠点击左右两端的图片来完成的。
{% img left /images/blog/apple_watch_guidelines/19.sliders_settings_brightness_2x.png 272 337 %}    

特点：  
1. 由中间的track和两边的两张图组成。  
2. 可以显示连续数据也可以显示离散数据。离散数据例如把大象装冰箱分三步，第一步在track上显示一条细格子，第二步显示两条，第三步显示三条。
3. 任何情况应该根据上次的设置加或减来设置新值
4. 并不会向用户显示内部数字化的值

注意事项：  
在两端放置恰当的图片让用户一眼就知道这个slider是做什么的。如果你不放，系统帮你放一个加号一个减号。  

## Maps

Map可以向用户展示地理位置。Map在第三方app内是一个静态截图，用户点击之后会跳转到系统的Maps app。

注意事项：  

1. 设置map region是应指定需要显示的位置的最小区域。为了展示出来，Map region需要在WatchKit extension中在运行时先行设置。选取的区域应包含全部需要展示的地理位置，也应当尽可能的小，这样来说对用户是最方便的。  
2. 不要为map对象指定超出content map区域的大小。Map应该在现有Apple Watch屏幕上不需要滚动就可以显示全部。  
3. 使用annotations在地图上高亮地点。Annotations是贴在map上的图片，用来标出地理位置，或者唤出此位置更多的信息。同一个map内的annotations应该限制在五个以内。  

Maps内建支持显示红、绿、紫三种颜色的大头钉。  
{% img left /images/blog/apple_watch_guidelines/20.map_pin_2x.jpg 402 387 %}  
你可以为annotations指定自定义图片，目标地点应该正对着图片底部的中心点。  
{% img left /images/blog/apple_watch_guidelines/21.map_image_2x.jpg 461 387 %}  

## Dates and Timers
Date对象和timer对象是一种特殊的label，用来展示和时间相关的信息。"这毕竟是块手表，我就给你封装个时间label吧。"    

### Date Labels

特点：  
1. 可以显示时间、日期，或者时间+日期
2. 可以设置多种显示格式、历法、时区
3. 不需要从WatchKit extension发起更新

任何需要显示当前日期或时间的地方，都要使用date对象

### Timer Labels
{% img left /images/blog/apple_watch_guidelines/21.map_image_2x.jpg 336 207 %}  

特点：
1. 从一个特定时间开始计时或者开始倒计时
2. 可以为计时选择多种显示格式
3. 不需要从WatchKit extension发起更新

任何需要显示计时的场景，都要使用timer label

## Menus
如果当前存在Menus，Force Touch事件将会换起它。Menus能够未当前屏幕设置几种相关操作，而且不唤起时不会占据屏幕空间。  
{% img left /images/blog/apple_watch_guidelines/23.menu_stopwatch_2x.png 272 337 %}  

注意事项：  
1.Menus可以显示一到四个actions。Actions按照向menu添加的顺序，从左到右，从上到下排布在屏幕上。Menus没有层级关系，也不能滚动。Menus可以在设计期（编译期）添加，也可在运行期添加。  
2. Menu的actions作用在当前一屏上，每一屏可以指定自己特定的menu或者不指定menu。Actions适用于整个屏幕，并不是为了某个特定的控件或者某块屏幕内的子区域准备的。（明明硬件上可以检测具体下压的点，苹果却没这么实现，可能体验不好？）  
3. 每个action应当有一张图片和一个label。Menu的图片应当是系统样式的线条风格，画在一个标准的背景上。Label限制两行以内，文案要尽可能短。  
4. Menus是可选实现的。仅当当前屏幕有相关操作时使用menu。如果不实现menu的话，当Force Touch触发时，系统将会显示一个提示动画。  

关于menu内使用的图片的更多信息，移步[Menu Images](https://developer.apple.com/library/prerelease/ios/documentation/UserExperience/Conceptual/WatchHumanInterfaceGuidelines/MenuImages.html#//apple_ref/doc/uid/TP40014992-CH18-SW1)。

# 图片与图标的设计规范

## 图标和图片尺寸

每个app都有一个独特的icon。和iPhone app不同，手表上home screen的app icon是不带title的，用户仅通过icon来区分app，所以icon不但要求是容易识别的，而且还应该能够表达出app的作用。  

{% img left /images/blog/apple_watch_guidelines/24.HomeScreen_2x.jpg 312 549 %}  

### 图标尺寸
Home screen上显示圆形图标。下表列出了各种情况下图标的正常尺寸。创建的icon文件是一张方形的图片，系统将自动将其切割成圆形。(注意Xcode中是以point为单位列出icon尺寸，非pixel为单位)

Asset|Apple Watch (38mm)|Apple Watch (42mm)
--------|--------|--------
Notification center icon|48 pixels|55 pixels
Home screen icon & Long look notification icon|80 pixels|88 pixels
Short Look icon|172 pixels|196 pixels

所有的图片都只创建@2x版本即可，无需创建其他尺寸的版本。  
所有的images和icons都应该使用png格式，但是要避免使用interlaced PNGs。  
Icons和images的标准颜色位深是24，红绿蓝各8位。也可附加一个8位的alpha通道，但是此通道不会背显示出来。也可以使用indexed colors PNGs来节约存储空间。  

### Home Screen Icon

Apple Watch上home screen的icon和iPhone上的相似，但是并没有标题。尽管Icon如此小，Icon也必须要清晰的表示app内容。如果WatchKit app与其对应的iOS功能接近，icon应该在视觉上相似。如果WatchKit app是iOS app的一个组件或是控制器的话，icon可以根据需要设计成不同的样式。  

注意事项：  
1. 为了最好的显示效果，你需要让专业的视觉设计师来设计图标。专业的设计师更知道该怎么专业的画图标。  
2. Icon要使用通用、易于理解的图像，避免使用晦涩、抽象、仅表达附带意义的图像作为icon。举个例子，Mail的icon是个信封，不是农村的铁邮箱，不是信使包，也不是邮局的标签。  
3. 尽量简洁。避免在icon里加很多各种各样的小图片。使用一个能表达app本质的元素，用简单、独特的形状绘制出来效果是最好的。给Icon添加细节时要谨慎。如果icon的内容或形状过于复杂，会让用户感觉迷惑，而且图标尺寸小的时候很容易发糊。  
4. Icon是app的主要概念的一个抽象表达。一般来说，用艺术的方式表达现实是一个好主意，因为这可以向用户强调app主题的抽象意义。  
5. 确保Apple Watch上的icon和iPhone上的icon相似，这会让用户自动把watch的app和iPhone上的app在脑海中相关联。  

请为两种尺寸的watch创建不同分辨率的home icon，这样才能让app在两种表盘大小上看起来都不错。Icon尺寸前面已经说过了。

## Menu Images

Force Touch menu中icon的alpha channel决定了icon的显示结果。其他颜色信息在显示时将被丢弃。  

图片画布的尺寸要比内容尺寸稍大。图标周围多余的空白部分可以确保在图标和menu icon的边缘之间有足够的边距。

Device|Canvas size|Content size
--------|--------|--------
Apple Watch (38mm)|70 pixels|46 pixels
Apple Watch (42mm)|80 pixels|54 pixels

设计menu icon的图形时，线的粗细要兼顾到屏幕尺寸和图形的复杂度。但线最少要保证4px宽，以避免显示发虚的情况。  
{% img left /images/blog/apple_watch_guidelines/25.menu_glyphs_2x.png 670 375 %}  
推荐使用PNG格式保存menu icon。  

Over
