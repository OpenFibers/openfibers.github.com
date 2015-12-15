---
layout: post
title: "Xcode插件"
date: 2015-10-04 11:38:06 +0800
comments: true
categories: ["Xcode"]
---

### 1. Alcatraz

其他插件的 package manager, 可以从此插件中搜索、安装、删除其他插件。  
[Github地址](https://github.com/supermarin/Alcatraz)

### 2. VVDocumenter-Xcode

///生成 Javadoc-Style 注释，可被 Xcode 识别，可生成文档。  
[Github地址](https://github.com/onevcat/VVDocumenter-Xcode)

### 3. MCLog

Log 的 filter，支持正则，响应式交互，尚有崩溃，不过值得一用。  
[Github地址](https://github.com/yuhua-chen/MCLog)  

### 4. Clang-Format

一键格式化脏乱差代码。还支持自动按段落排序#import "headers"。  
安装后在工程中建立 .clang-format 文件，在其中配置代码格式，再使用此插件格式化代码。  
前几天发pr修复了和 oh-my-zsh 同时使用的一处 crash，作者刚刚merge了，建议更新到最新。  
推荐格式：  

```
BasedOnStyle: WebKit
BreakBeforeBraces: Allman
PointerAlignment: Right
IndentCaseLabels: true
AlignAfterOpenBracket: true
```

[Github地址](https://github.com/travisjeffery/ClangFormat-Xcode)

### 5. FuzzyAutocomplete

代码自动补全的增强工具，输入反应稍微减慢，不装其他输入增强工具时卡顿尚可接受，值得一用。  
[Github地址](https://github.com/FuzzyAutocomplete/FuzzyAutocompletePlugin)