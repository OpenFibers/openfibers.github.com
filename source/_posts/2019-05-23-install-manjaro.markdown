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

### 设置国内源  

华科源（比清华的快很多），更改信任设置（以便安装 chrome、搜狗拼音、微信、QQ 等软件）  

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
sudo pacman -Syy && sudo pacman -S archlinuxcn-keyring
```

切换默认源：  

```
sudo pacman-mirrors -i -c China -m rank # 弹框后选择华科的源，点击确认
```

### 系统更新  

```
sudo pacman -Syu
```

然后打开 Octopi，系统更新应该没有了

### 看看有没有没驱动的硬件

### Chrome

```
sudo pacman -S google-chrome
```

### Google 拼音

```
sudo pacman -S fcitx-im     # 全部安装
sudo pacman -S fcitx-configtool
sudo pacman -S fcitx-googlepinyin
```

然后`vi ~/.xprofile`，加入：  

```
export LC_ALL=zh_CN.UTF-8
export GTK_IM_MODULE=fcitx
export QT_IM_MODULE=fcitx
export XMODIFIERS=”@im=fcitx”
```

注销重新登录，打开 Fcitx Configuration，启用 google pinyin。这时候语言会变成中文，如果想换回原语言，注释掉 `export LC_ALL=zh_CN.UTF-8`，注销再登录回来即可。

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

安装：  

```
sudo pacman -S electronic-wechat
```

启动：  

```
electronic-wechat
```

### Vim

```
sudo pacman -S vim
```

### yaourt

```
sudo pacman -S yaourt
```


### Sublime Text 3

安装：  

```
curl -O https://download.sublimetext.com/sublimehq-pub.gpg && sudo pacman-key --add sublimehq-pub.gpg && sudo pacman-key --lsign-key 8A8F901A && rm sublimehq-pub.gpg
echo -e "\n[sublime-text]\nServer = https://download.sublimetext.com/arch/stable/x86_64" | sudo tee -a /etc/pacman.conf
sudo pacman -Syu sublime-text
```

<!-- 
3207 激活

1. subl /etc/hosts 加入 

127.0.0.1 license.sublimehq.com
127.0.0.1 45.55.255.55
127.0.0.1 45.55.41.223
0.0.0.0 license.sublimehq.com
0.0.0.0 45.55.255.55
0.0.0.0 45.55.41.223
127.0.0.1 www.sublimetext.com
127.0.0.1 sublimetext.com
127.0.0.1 sublimehq.com
127.0.0.1 telemetry.sublimehq.com

2. HEX打开 
搜索16进制 97 94 0D 00
改为  00 00 00 00

3. 禁用更新
Go to “Preferences -> Settings-User/Distraction Free“
Add this line: “update_check”: false,

4. 输入 license

----- BEGIN LICENSE -----
TwitterInc
200 User License
EA7E-890007
1D77F72E 390CDD93 4DCBA022 FAF60790
61AA12C0 A37081C5 D0316412 4584D136
94D7F7D4 95BC8C1C 527DA828 560BB037
D1EDDD8C AE7B379F 50C9D69D B35179EF
2FE898C4 8E4277A8 555CE714 E1FB0E43
D5D52613 C3D12E98 BC49967F 7652EED2
9D2D2E61 67610860 6D338B72 5CF95C69
E36B85CC 84991F19 7575D828 470A92AB
------ END LICENSE ------

 -->

### Python3 + Pycharm

```
sudo pacman -S pycharm-community-edition
```

清华 pypi 源：  

```
pip3 config set global.index-url https://pypi.tuna.tsinghua.edu.cn/simple
```

### IntelliJ IDEA

```
sudo pacman -S intellij-idea-community-edition
```

### shadowsocks

```
sudo pacman -S shadowsocks-libev
sudo pacman -S shadowsocks-qt5
```

然后 chrome 在 extension store 安装 Proxy SwitchyOmega，并配置。

### 字体

小撸怡情，不然舔到最后一无所有

```
sudo pacman -S noto-fonts noto-fonts-cjk noto-fonts-emoji
yaourt -S ttf-meslo
yaourt -S cairo-coloredemoji
fc-cache --force --verbose
```

### 蓝牙

没搞定
