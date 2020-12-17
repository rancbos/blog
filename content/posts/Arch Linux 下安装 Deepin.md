---
title: "Arch Linux 下安装 Deepin"
date: 2020-12-17T17:42:35+08:00
tags: 
- ArchLinux
- Deepin
- Install
categories: 
- ArchLinux
- Deepin
draft: false
---

### 安装Xorg

```
pacman -S xorg xorg-server
```

### 安装桌面

其中`deepin`是必须的`deepin-extra`是一些常用软件包括终端和文件管理器等（为了方便推荐安装上）

```
pacman -S deepin deepin-extra
```

### `lightdm`

```
pacman -S lightdm
```

修改 lightdm 配置文件

```
vim /etc/lightdm/lightdm.conf
```

指定 `greeter-session` 为 `lightdm-deepin-greeter`

```
[Seat:*]
...
greeter-session=lightdm-deepin-greeter
```

开机自启

```
systemctl enable lightdm.service
```

### 安装声音

```
sudo pacman -S alsa-utils pulseaudio pulseaudio-alsa
```

### 一些软件的安装

安装 Deepin Terminal 需要用到的软件包

```
sudo pacman -S zssh lrzsz
```

### 遇到的问题

无法更换壁纸，解决方法

```
sudo pacman -S deepin-kwin #一个窗口管理器
```