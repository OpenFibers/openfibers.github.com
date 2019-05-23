---
layout: post
title: "Install Manjaro from macOS"
date: 2019-05-23 11:09:28 +0800
comments: true
categories: 
---

### 从 macOS 10.14 制作安装 U 盘

1. 下载好 iso 之后，将 iso 改名为 ~/Downloads/manjaro-kde.iso
2. 然后将 iso 转换成 dmg:  

```
hdiutil convert -format UDRW -o ~/Downloads/manjaro ~/Downloads/manjaro-kde.iso
```

3. 看一下 U 盘是 disk几：

```
diskutil list
```

4. 假设 U 盘是 disk9，分区，unmount，再写入镜像

```
diskutil partitionDisk disk9 1 GPT HFS+ newdisk R
diskutil unmountDisk /dev/disk9
sudo dd if=/Users/openthread/Downloads/manjaro.dmg of=/dev/rdisk9 bs=1m
```

5. U盘插到 pc，重启，安装

### 安装好以后

系统更新：

```
sudo pacman -Syu
```
