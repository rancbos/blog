---
title: "Arch Linux 下软件安装"
date: 2020-12-17T16:19:02+08:00
tags: 
- ArchLinux
- Install
categories: 
- ArchLinux
draft: false
---

## Fonts

```
yaourt -S nerd-fonts-complete
sudo pacman -S  wqy-zenhei
sudo pacman -S ttf-roboto noto-fonts noto-fonts-cjk adobe-source-han-sans-cn-fonts adobe-source-han-serif-cn-fonts ttf-dejavu
```

## yay & yaourt

```
sudo pacman -S yaourt
```

[**yay**](https://github.com/Jguer/yay)

```
pacman -S --needed git base-devel
git clone https://aur.archlinux.org/yay.git
cd yay
makepkg -si
```

**命令**

yay 兼容 Pacman 的命令行参数。

```
## pacman
pacman -Syu # 同步数据包后更新系统

pacman -Sy 包名 # 同步包数据库后安装。

pacman -R 包名 # 删除包不删除依赖
pacman -Rs 包名 # 删除包的同时删除不被其它包使用的依赖
pacman -Rd 包名 # 删除包不检查依赖

pacman -Ss 关键字 # 搜索含关键字的包。
pacman -Qi 包名 # 查看有关包的信息。

pacman -Sc Pacman #清理未安装的包文件
## yay
yay -Su # 升级软件库
yay -Sy # 更新软件库
yay -Syy # 强制更新软件库
yay -Sy 包名 # 安装命令
yay -Rs 包名 # 卸载命令
yay -Ss 关键字 # 搜索命令
```

PS：在软件中心里构建 AUR 程序其实就是相当于帮我们做了手动 `makepkg`  的工作。

## Vim

Archlinux Wiki：[Vim](https://wiki.archlinux.org/index.php/Vim_(简体中文))

```
sudo pacman -S gvim
```

vim 在 sudo 下使用当前用户的配置

```
cd /etc
sudo chmod u+w sudoers
sudo vim sudoers

Defaults env_keep += "HOME"  # 取消注释
```

## Neovim

Archlinux Wiki：[Neovim](https://wiki.archlinux.org/index.php/Neovim)

```
sudo pacman -S neovim
```

**[vim-plug](https://github.com/junegunn/vim-plug)** 

**Unix, Linux**

```
sh -c 'curl -fLo "${XDG_DATA_HOME:-$HOME/.local/share}"/nvim/site/autoload/plug.vim --create-dirs \
       https://raw.githubusercontent.com/junegunn/vim-plug/master/plug.vim'
```

**Linux (Flatpak)**

```
curl -fLo ~/.var/app/io.neovim.nvim/data/nvim/site/autoload/plug.vim \
    https://raw.githubusercontent.com/junegunn/vim-plug/master/plug.vim
```

## Visusl Studio Code

Archlinux Wiki：[Visual Studio Code](https://wiki.archlinux.org/index.php/Visual_Studio_Code_(简体中文))

```
sudo pacman -S code
```

## Fcitx

输入法引擎见这：[Fcitx 输入法引擎](https://wiki.archlinux.org/index.php/Fcitx_(简体中文))

我选用的是 [fcitx-sogoupinyin](https://aur.archlinux.org/packages/fcitx-sogoupinyin/) 。

**安装** **`fcitx`** 

```
sudo pacman -S fcitx
sudo pacman -S fcitx-configtool # 配置工具
yay -S fcitx-sogoupinyin
```

**不能输入中文**

建议查看：[Fcitx_(简体中文)#输入法模块](https://wiki.archlinux.org/index.php/Fcitx_(简体中文)#输入法模块) 里的设置环境变量。

修改 `.xprofile`  （若无则新建），添加如下配置。

```
export GTK_IM_MODULE=fcitx
export QT_IM_MODULE=fcitx
export XMODIFIERS=@im=fcitx
```

然后重启之。

## Zsh

> [Zsh](https://wiki.archlinux.org/index.php/Zsh_(简体中文)) 是一款功能强大的 Shell 软件，既可以作为交互式终端来使用，也可以作为脚本语言解释器来使用。它在兼容 POSIX 的 sh 的同时（默认不兼容，仅在使用 `emulate sh` 时兼容），还改进了 Tab 补全和通配符等功能。

查看当前环境 shell

```
echo $SHELL
```

首先：安装官方源的 zsh，默认安装位置: `~/.zshrc`

```
sudo pacman -S zsh
```

知道你的系统有几种 shell，可以通过以下命令查看：

```
cat /etc/shells
```

设为默认 shell

```
chsh -s /bin/zsh
or usermod -s /bin/zsh JW
```

当然你实在不愿意把 zsh 当成默认的 shell, 而又想使用它, 那么你可以每次进入是都使用 `zsh` 进 入, 而输入 `exit` 退出

这个命令里面的 `jw`  是系统用户名，在执行这条命令的时候，切记换成自己的用户名。用这个命令的缺点是，只能更改单个用户的，不能修改所有用户的。

[**zinit** ](https://github.com/zdharma/zinit)

```
mkdir ~/.zinit
git clone https://github.com/zdharma/zinit.git ~/.zinit/bin
```

主题：[powerlevel10k](https://github.com/romkatv/powerlevel10k)

## fzf

```
sudo pacman -S fzf
```

## Trash

首先，最需要说明的是，在 Linux 系统中，`rm` 是个非常非常可怕的指令。一来该指令非常简短、二来我们使用系统，总免不了要释放空间，删除掉不要的旧文件。

而使用 `rm` 删除文件，若是不小心误删了，我们可能只能赶快藉由 Inode 还原 —— 不过这也是常常救不回来了，更别提连 Inode 都释放掉的情况。

而 [trash](https://github.com/andreafrancia/trash-cli)，就是一个希望避免这种事情发生，可用于取代 `rm` 的指令 (不过无法完全取代) —— `trash`。

`trash` 是什么样的指令呢？简单来讲，大部分的系统都有个专门的规范，将欲删除掉的文件放在一个暂时存放的地方，像是 Windows 就称其为『回收站』，而在 Linux 中，则就是 Trash。

一般我们在桌面环境中使用 del 键 删除档案，档案会自动移动至 Trash 底下。等到我们确定真的不要这个档案了，我们再进入 Trash 中将其彻底删除。

而 `trash` 指令的操作流程也是一样。使用这个指令删除的档案不会彻底消失，只是暂时移动到了 Trash 底下，随时可以还原、删除。这样一来，就避免了 `rm` 误删档案的问题了。

**安装**

在使用 `trash` 指令前，我们需要先安装它。在 Archlinux 系统中，我们可以使用以下指令安装：

```
sudo pacman -S trash-cli
```

安装结束以后，我们就可以开始使用 `trash` 指令了。

**trash 的几个常用指令**

**1. 使用 trash-list 查看 Trash 内有无删除的档案**

```
trash-list
```

不过一开始，若是没有任何档案的话，这里便不会显示任何资讯。

**2. 使用 trash 删除档案**

假设我有两个想要删除的档案，分别叫做 01.txt 以及 02.txt。

```
trash 01.txt 02.txt
```

使用空格分开两个档案，可以看到这两个档案已经不在当前资料夹底下。这时，再使用 `trash-list` 确认档案已经在 Trash 底下，应该会看到像是这样的资讯：

```
2020-08-04 20:38:40 /home/clay/02.txt
2020-08-04 20:38:40 /home/clay/01.txt
```

纪录着删除的时间、以及删除前的档案路径。

**3. 使用 restore-trash 还原档案**

如果要还原 Trash 内的档案，可以输入：

```
restore-trash
```

会跳出这样的画面：

```
restore-trash
0 2020-08-04 20:38:40 /home/clay/02.txt
1 2020-08-04 20:38:40 /home/clay/01.txt
What file to restore [0..1]:
```

假设我想要还原 02.txt，那么我就要输入它的编号 0。

这在还原大量档案的时候稍微有点麻烦 …… 其实可以藉由 `cp` 指令直接从 ~/.local/share/Trash/files/ 路径底下复制出来。这里建议不要使用 `mv` 指令直接搬移，或造成之后 `trash` 指令的混乱。

**4. 使用 trash-empty** **清空 Trash 内的所有档案**

很简单，直接使用：

```
trash-empty
```

就会将 Trash 底下的所有档案删除了。不过跟 `rm` 指令一样，要先确认里面都是不要的档案。

到头来，我觉得就算有再多的保险步骤，会不小心删错的可能性一直都会在。最重要的是，使用者要仔细确认自己所下的指令，这才是治本的方法。

## Wps

```
yaourt -S wps-office
yay -S ttf-wps-fonts
```

## 网易云音乐

```
yay -S netease-cloud-music
```

## 有道词典

```
yay -S youdao-dict
```

## Vlc

```
sudo pacman -S vlc
```

## Typora

```
sudo pacman -S typora
```

## Kitty

```
sudo pacman -S kitty
```

快捷键

[**Scrolling**](https://sw.kovidgoyal.net/kitty/#id9)

| Action           | Shortcut                                                     |
| ---------------- | ------------------------------------------------------------ |
| Scroll line up   | [`ctrl+shift+up`](https://sw.kovidgoyal.net/kitty/conf.html#shortcut-kitty.Scroll-line-up) (also ⌥+⌘+⇞ and ⌘+↑ on macOS) |
| Scroll line down | [`ctrl+shift+down`](https://sw.kovidgoyal.net/kitty/conf.html#shortcut-kitty.Scroll-line-down) (also ⌥+⌘+⇟ and ⌘+↓ on macOS) |
| Scroll page up   | [`ctrl+shift+page_up`](https://sw.kovidgoyal.net/kitty/conf.html#shortcut-kitty.Scroll-page-up) (also ⌘+⇞ on macOS) |
| Scroll page down | [`ctrl+shift+page_down`](https://sw.kovidgoyal.net/kitty/conf.html#shortcut-kitty.Scroll-page-down) (also ⌘+⇟ on macOS) |
| Scroll to top    | [`ctrl+shift+home`](https://sw.kovidgoyal.net/kitty/conf.html#shortcut-kitty.Scroll-to-top) (also ⌘+↖ on macOS) |
| Scroll to bottom | [`ctrl+shift+end`](https://sw.kovidgoyal.net/kitty/conf.html#shortcut-kitty.Scroll-to-bottom) (also ⌘+↘ on macOS) |

[**Tabs**](https://sw.kovidgoyal.net/kitty/#id10)

| Action            | Shortcut                                                     |
| ----------------- | ------------------------------------------------------------ |
| New tab           | [`ctrl+shift+t`](https://sw.kovidgoyal.net/kitty/conf.html#shortcut-kitty.New-tab) (also ⌘+t on macOS) |
| Close tab         | [`ctrl+shift+q`](https://sw.kovidgoyal.net/kitty/conf.html#shortcut-kitty.Close-tab) (also ⌘+w on macOS) |
| Next tab          | [`ctrl+shift+right`](https://sw.kovidgoyal.net/kitty/conf.html#shortcut-kitty.Next-tab) (also ^+⇥ and ⇧+⌘+] on macOS) |
| Previous tab      | [`ctrl+shift+left`](https://sw.kovidgoyal.net/kitty/conf.html#shortcut-kitty.Previous-tab) (also ⇧+^+⇥ and ⇧+⌘+[ on macOS) |
| Next layout       | [`ctrl+shift+l`](https://sw.kovidgoyal.net/kitty/conf.html#shortcut-kitty.Next-layout) |
| Move tab forward  | [`ctrl+shift+.`](https://sw.kovidgoyal.net/kitty/conf.html#shortcut-kitty.Move-tab-forward) |
| Move tab backward | [`ctrl+shift+,`](https://sw.kovidgoyal.net/kitty/conf.html#shortcut-kitty.Move-tab-backward) |
| Set tab title     | [`ctrl+shift+alt+t`](https://sw.kovidgoyal.net/kitty/conf.html#shortcut-kitty.Set-tab-title) (also ⇧+⌘+i on macOS) |

[**Windows**](https://sw.kovidgoyal.net/kitty/#id11)

| Action                | Shortcut                                                     |
| --------------------- | ------------------------------------------------------------ |
| New window            | [`ctrl+shift+enter`](https://sw.kovidgoyal.net/kitty/conf.html#shortcut-kitty.) (also ⌘+↩ on macOS) |
| New OS window         | [`ctrl+shift+n`](https://sw.kovidgoyal.net/kitty/conf.html#shortcut-kitty.New-OS-window) (also ⌘+n on macOS) |
| Close window          | [`ctrl+shift+w`](https://sw.kovidgoyal.net/kitty/conf.html#shortcut-kitty.Close-window) (also ⇧+⌘+d on macOS) |
| Next window           | [`ctrl+shift+\]`](https://sw.kovidgoyal.net/kitty/conf.html#shortcut-kitty.Next-window) |
| Previous window       | [`ctrl+shift+[`](https://sw.kovidgoyal.net/kitty/conf.html#shortcut-kitty.Previous-window) |
| Move window forward   | [`ctrl+shift+f`](https://sw.kovidgoyal.net/kitty/conf.html#shortcut-kitty.Move-window-forward) |
| Move window backward  | [`ctrl+shift+b`](https://sw.kovidgoyal.net/kitty/conf.html#shortcut-kitty.Move-window-backward) |
| Move window to top    | [`ctrl+shift+``](https://sw.kovidgoyal.net/kitty/conf.html#shortcut-kitty.Move-window-to-top) |
| Focus specific window | [`ctrl+shift+1`](https://sw.kovidgoyal.net/kitty/conf.html#shortcut-kitty.First-window), [`ctrl+shift+2`](https://sw.kovidgoyal.net/kitty/conf.html#shortcut-kitty.Second-window) ... [`ctrl+shift+0`](https://sw.kovidgoyal.net/kitty/conf.html#shortcut-kitty.Tenth-window) (also ⌘+1, ⌘+2 ... ⌘+9 on macOS) (clockwise from the top-left) |

## Flameshot

**介绍**

Archlinux Wiki：[Flameshot](https://wiki.archlinux.org/index.php/Flameshot)

Flameshot 是一个截屏程序。它有一个交互式的图形用户界面和控件，可以选择所需的捕获区域，移动和调整捕获窗口的大小，使用常见的绘图工具(铅笔、线条、矩形、圆形、模糊、撤销/重做)进行编辑，并选择输出目的地(复制到剪贴板，保存到磁盘，上传到 Imgur，用另一个程序打开)。

**安装**

```
sudo pacman -S flameshot
```

命令 截图： `flameshot gui` 

​    配置： `flameshot config` 

## Ranger

```
sudo pacman -S ranger
```

## p7zip

```
sudo pacman -S p7zip
```

## Chrome

```
yay -S google-chrome
```

## Bat

```
sudo pacman -S bat
```

## Lsd

```
sudo pacman -S lsd
```