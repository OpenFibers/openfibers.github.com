---
layout: post
title: "Install Manjaro from macOS"
date: 2019-05-23 11:09:28 +0800
comments: true
categories: 
---

## 从 macOS 10.14 制作安装 U 盘

下载好 iso 之后，将 iso 改名为 ~/Downloads/manjaro-kde.iso，然后将 iso 转换成 dmg:  

```
hdiutil convert -format UDRW -o ~/Downloads/manjaro ~/Downloads/manjaro-kde.iso
```

看一下 U 盘是 disk几：  

```
diskutil list
```

假设 U 盘是 disk9，分区，unmount，再写入镜像：  

```
diskutil partitionDisk disk9 1 GPT HFS+ newdisk R
diskutil unmountDisk /dev/disk9
sudo dd if=/Users/openthread/Downloads/manjaro.dmg of=/dev/rdisk9 bs=1m
```

U盘插到 pc，重启，安装

<!--more-->

## 安装好以后

### 系统更新：  

```
sudo pacman -Syu
```

然后打开 Octopi，系统更新应该没有了

### 看看有没有没驱动的硬件

### 设置国内源：  

清华源：  

```
sudo pacman-mirrors -i -c China -m rank # 弹框后选择清华的源，确认
```

华科源，更改信任设置（以便安装 chrome、搜狗拼音、微信、QQ 等软件）  

```
sudo vi /etc/pacman.conf
```

加入：  

```
[archlinuxcn]
SigLevel = Optional TrustedOnly
Server = https://mirrors.ustc.edu.cn/archlinuxcn/$arch
```

更新软件缓存，更新 keyring：  

```
sudo pacman -Syy && sudo pacman-S archlinuxcn-keyring
```

### Chrome

```
sudo pacman-S google-chrome
```

### 搜狗拼音

```
sudo pacman -S fcitx-im     # 全部安装
sudo pacman -S fcitx-configtool
sudo pacman -S fcitx-sogoupinyin
```

然后`vi ~/.xprofile`，加入：  

```
export GTK_IM_MODULE=fcitx
export QT_IM_MODULE=fcitx
export XMODIFIERS=”@im=fcitx”
```

重启后 ctrl + 空格 应该能切换输入法了。

### 网易云音乐  

安装：  
```
sudo pacman -S netease-cloud-music
```

启动：

```
netease-cloud-music
```

### Dota2

直接用自带的 steam 装就完事

### 微信

### Sublime Text 2

### Pycharm

### IntelliJ IDEA

