---
layout: post
title: "日了狗的BitCode"
date: 2016-01-14 18:06:54 +0800
comments: true
categories: ['iOS', 'Xcode', 'BitCode']
---

苹果对BitCode的介绍看上去很美，今天尝试了一下，掉入深坑，克服种种困难终于爬上来了。  
对于App开发者，建议在苹果做出改进之前暂时不要使用此技术。友情提示，现在最新版本iOS为9.2，Xcode为7.2。  

下面介绍一下如何开启BitCode，以及我掉的坑有哪些。  

<!--more-->

### 1. App工程开启BitCode

在App的Target->Build Settings->Build Options中找到Enable BitCode，设置为YES，然后顶栏菜单Product->Archive。  

第一次使用BitCode编译很可能出现链接错误，ld时提示有文件编译时未开启bit code。比如下面这条错误：

ld: 'FooFile(FooFile.o)' does not contain bitcode. You must rebuild it with bitcode enabled (Xcode setting ENABLE_BITCODE), obtain an updated library from the vendor, or disable bitcode for this target.   

ld会提示什么framework或.a文件编译时未开启BitCode，这些framework或.a文件需要开启BitCode重新编译后再添加到工程，然后再次尝试编译。  

如果编译成功，上传到iTunes Connect后却提示如下错误：  

{% img left /images/blog/fucking_bitcode/1.iTunesConnectError.png 509 379 %}  

则可能由于App不是用Archive编译的，或者在Custom Compiler Flags中未加入-fembed-bitcode。  

### 2. 关于-fembed-bitcode

BitCode是编译期的feature，而非链接期的feature，也就是编译过程中每个.o文件都会有一个叫做__bitcode的段落生成。  

在Build Options中启用BitCode，且使用Build而非Archive编译时，Xcode会自动添加编译选项-fembed-bitcode-marker，这个选项的意思大概就是说：如果BitCode开启的话，这里本来应当是放bitcode的，实际上没放。  

在Build Options中启用BitCode，且使用Archive编译时，Xcode会自动添加编译选项-fembed-bitcode，此时才是真正开启了BitCode。  
如果编译选项设置-fembed-bitcode-marker，编译成功后上传iTunes Connect，就会出现1.中图片的错误。当然这里iTunes Connect的体验是非常差的，报个错还不告诉原因：“开发者你有本事猜猜看呀？”  

如果使用Build编译想强制开启-fembed-bitcode，只需在Target->Build Settings->Custom Compiler Flags中加入-fembed-bitcode即可。  

相关讨论可以参考[How do I xcodebuild a static library with Bitcode enabled?](http://stackoverflow.com/questions/31486232/how-do-i-xcodebuild-a-static-library-with-bitcode-enabled)

### 3. 创建支持 BitCode 的 Framework/Static Library

创建支持BitCode，并同时支持模拟器和真机的Framework/Static Library，和之前并无太大区别。  

Xcode6开始，在创建新工程时，增加了选项iOS->Framework and Library->Cocoa Touch Framework。假设想创建framework，可以首先选择这个选项，新建一个iOS framework project。如果想创建static library，则选择"Cocoa Touch Static Library"。  
创建完project，第一件事是在Target->Build Settings->Build Options中开启Enable BitCode，然后去Target->Build Settings->Custom Compiler Flags中加入-fembed-bitcode。因为一般来说，创建framework或static library，都是需要同时兼容模拟器和真机的，而Xcode并未提供便利的一键生成同时兼容模拟器和真机的executable binary的方式，而是要自己手动合并模拟器和真机的framework/static library。而在编译到模拟器时，是不支持Archive的，所以只有加上-fembed-bitcode选项才能支持BitCode。  
然后，在Build Settings->Architectures添加额外的真机指令集支持，在当前（2016.01.15）来说，就是再加上个armv7s。  

编译选项设置好之后，和以往一样，添加好各种代码文件和资源文件，将build configuration 设为 release，然后分别编译到模拟器和真机。之后手动合并fat executable file:  

合并framework的话：
```bash
	#combine framework
	cd $product_folder
	lipo -create -output "FooFramework" "Release-iphoneos/FooFramework.framework/FooFramework" "Release-iphonesimulator/FooFramework.framework/FooFramework"
	lipo -info FooFramework
	cp -R Release-iphoneos/FooFramework.framework ./FooFramework.framework
	mv FooFramework ./FooFramework.framework/FooFramework
```

lipo -info 后，对于常规的framework往往应该是如下显示：  

```
Architectures in the fat file: FooFramework are: i386 x86_64 armv7 armv7s arm64 
```

合并static library的话：
```
	#combine library
	cd $product_folder
	lipo -create -output "libFooLibrary.a" "Release-iphoneos/libFooLibrary.a" "Release-iphonesimulator/libFooLibrary.a" 
	lipo -info libFooLibrary.a
```

lipo -info 后，对于常规的static library往往应该是如下显示：  

```
Architectures in the fat file: libFooLibrary.a are: i386 x86_64 armv7 armv7s arm64 
```

然后将 Framework 或（static library 和 public headers）加入到App工程中，开启App Target的BitCode，Archive编译，上传iTunes Connect测试是否能正常处理。如果是Framework的话，还需在App Target->General->Embedded Binaries中加入此Framework，否则会出现error。   

### 4. 包的体积不降反增

花了这么大力气，总算成功上传Test Flight了，却发现原本31.7M的包体积变成了64M，却不知为何。一口老血喷了出来。所以你如果看完了本文，还要坚持尝试Bit Code的话，请做好充分的心理准备。  

Over