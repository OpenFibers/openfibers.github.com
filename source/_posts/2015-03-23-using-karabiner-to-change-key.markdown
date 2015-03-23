---
layout: post
title: "如何优雅的使用60%键盘----使用Karabiner将control+P、N、B、F修改为上下左右"
date: 2015-03-23 11:12:37 +0800
comments: true
categories: ["键盘", "OSX"]
---

最近新宠是各种GH60，刷固件修改按键颇为不便，软件修改就方便多了。  
经常有软件不支持Control+P、N、B、F做上下左右操作，于是想到直接把Control+P、N、B、F改为上下左右。  

OSX上可以用[Karabiner](https://pqrs.org/osx/karabiner/)自定义按键，可定制程度非常高，其前身是Keymap4MacBook，[已开源在https://github.com/tekezo/Karabiner](https://github.com/tekezo/Karabiner)。  

首先从官网下载一个最新版并安装好。我需要将将control+P、N、B、F修改为上下左右，在预置中找不到相应设置。直接在Misc & Uninstall页面点击Open private.xml，修改这个配置文件可以完成各种高度可定制化改键。  

<!--more-->

打开private.xml填入以下内容：  

```xml
<?xml version="1.0"?>
<root>
    <item>
        <name>Control+P to Arrow Up</name>
        <appendix>ctrl+p, send up</appendix>
        <identifier>private.controlPToUp</identifier>
        <autogen>
            --KeyToKey--
            KeyCode::P, ModifierFlag::CONTROL_L,
            KeyCode::CURSOR_UP
        </autogen>
    </item>
    <item>
        <name>Control+N to Arrow Down</name>
        <appendix>ctrl+n, send down</appendix>
        <identifier>private.controlNToDown</identifier>
        <autogen>
            --KeyToKey--
            KeyCode::N, ModifierFlag::CONTROL_L,
            KeyCode::CURSOR_DOWN
        </autogen>
    </item>
    <item>
        <name>Control+B to Arrow Left</name>
        <appendix>ctrl+b, send left</appendix>
        <identifier>private.controlBToLeft</identifier>
        <autogen>
            --KeyToKey--
            KeyCode::B, ModifierFlag::CONTROL_L,
            KeyCode::CURSOR_LEFT
        </autogen>
    </item>
    <item>
        <name>Control+F to Arrow Right</name>
        <appendix>ctrl+f, send right</appendix>
        <identifier>private.controlFToRight</identifier>
        <autogen>
            --KeyToKey--
            KeyCode::F, ModifierFlag::CONTROL_L,
            KeyCode::CURSOR_RIGHT
        </autogen>
    </item>
</root>
```

然后在Change Key页面点击Reload XML。如果XML没有编译错误，则自定义的remapping出现在列表最顶部。勾选即可生效。  

另感觉FC660M上Shift+Esc输出"~"十分人性化，顺便加一发：  

```
    <item>
        <name>Shift+Esc to ~</name>
        <appendix>Shift+Esc, send ~</appendix>
        <identifier>private.shiftEscToBackquote</identifier>
        <autogen>
            --KeyToKey--
            KeyCode::ESCAPE, ModifierFlag::SHIFT_L,
            KeyCode::BACKQUOTE, ModifierFlag::SHIFT_L,
        </autogen>
    </item>
```

Over
