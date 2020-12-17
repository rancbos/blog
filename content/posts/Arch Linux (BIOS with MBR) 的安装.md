---
title: "Arch Linux (BIOS With MBR) 的安装"
date: 2020-12-17T16:56:20+08:00
tags: 
- ArchLinux
- Install
categories: 
- ArchLinux
draft: false
---

## 安装前的准备

### 验证启动模式

要验证启动模式，请用下列命令列出 [efivars](https://wiki.archlinux.org/index.php/Efivars) 目录：

```
ls /sys/firmware/efi/efivars
```

如果命令没有错误地显示了目录，则系统以 UEFI 模式启动。 如果目录不存在，系统可能以 [BIOS](https://en.wikipedia.org/wiki/BIOS) 模式 (或 [CSM](https://en.wikipedia.org/wiki/Compatibility_Support_Module) 模式) 启动。如果系统未以您想要的模式引导启动，请参考您的主板手册。

我的笔记本电脑型号是：ThinkPad E540。型号比较老旧，需要使用 BIOS 模式启动。

确认主板系统是 BIOS，这里使用 MBR 分区格式，关于究竟该使用 MBR 还是 GPT 请参考[这里](https://wiki.archlinux.org/index.php/Partitioning_(简体中文)#选择_GPT_还是_MBR)。

### 制作启动盘

ArchLinux 下载地址：https://www.archlinux.org/download/

写入工具推荐：[Rufus](https://rufus.ie/)



### 连接到因特网

用下面步骤设置网络：



- 确保系统已经启用了 [网络接口](https://wiki.archlinux.org/index.php/Network_configuration#Network_interfaces)，用 [ip-link(8)](https://jlk.fjfi.cvut.cz/arch/manpages/man/ip-link.8) 检查: `ip link` ，并启动网卡 `ip link set wlan0 up` ，wlan0 为要启动的网络接口名称

- 对于无线网络，请确保无线网卡未被 [rfkill](https://wiki.archlinux.org/index.php/Rfkill) 禁用。
- 要连接到网络:

- - 有线以太网 —— 连接网线
  - WiFi —— 使用 [iwctl](https://wiki.archlinux.org/index.php/Iwctl) 验证无线网络（可选）



（1）查找所有的无线网络

```
iwlist wlan0 scan | grep ESSID
```

注：wlan0 为要启动的网络接口名称



（2）使用 `wpa_passphrase` 和 `wpa_supplicant` 连接网络

```
wpa_passphrase 网络 密码 > 文件名
```

eg： `wpa_passphrase wang 123456 > internet.conf` 

```
vim internet.conf # 查看网络名称和密码是否正确
wpa_supplicant -c internet.conf -i wlan0 &
```

这时还不能连接网络，需要 [dhcpcd](https://wiki.archlinux.org/index.php/Dhcpcd) 分配动态地址。

```
dhcpcd &
```

- 用 [ping](https://en.wikipedia.org/wiki/ping_(networking_utility)) 检查网络连接:

```
ping baidu.com
```



### 更新系统时间

使用 [timedatectl(1)](https://jlk.fjfi.cvut.cz/arch/manpages/man/timedatectl.1) 确保系统时间是准确的：

```
timedatectl set-ntp true
```

可以使用 timedatectl status 检查服务状态。



### 建立硬盘分区

磁盘若被系统识别到，就会被分配为一个[块设备](https://en.wikipedia.org/wiki/zh:设备文件系统#.E5.91.BD.E5.90.8D.E7.BA.A6.E5.AE.9A)，如 `/dev/sda`, `/dev/nvme0n1` 或 `/dev/mmcblk0`。可以使用 [lsblk](https://wiki.archlinux.org/index.php/Lsblk) 或者 *fdisk* 查看：

```
fdisk -l
```

对于一个选定的设备，以下的*分区*是必须要有的：

- 一个根分区（挂载在 [根目录](https://en.wikipedia.org/wiki/Root_directory)）`/`；
- 要在 [UEFI](https://wiki.archlinux.org/index.php/UEFI) 模式中启动，还需要一个 [EFI 系统分区](https://wiki.archlinux.org/index.php/EFI_system_partition)。

如果需要创建多级存储例如 [LVM](https://wiki.archlinux.org/index.php/LVM)、[disk encryption](https://wiki.archlinux.org/index.php/Disk_encryption) 或 [RAID](https://wiki.archlinux.org/index.php/RAID)，请在此时完成。



#### 分区示例

BIOS 与 [MBR](https://wiki.archlinux.org/index.php/Partitioning_(简体中文)#Master_Boot_Record)

| 挂载点   | 分区          | [分区类型](https://en.wikipedia.org/wiki/Partition_type) | 建议大小     |
| -------- | ------------- | -------------------------------------------------------- | ------------ |
| `/mnt`   | `/dev/sd*X*1` | Linux                                                    | 剩余空间     |
| `[SWAP]` | `/dev/sd*X*2` | Linux swap (交换空间)                                    | 大于 512 MiB |



UEFI 与 [GPT](https://wiki.archlinux.org/index.php/Partitioning_(简体中文)#GUID_分区表)

| 挂载点                    | 分区          | [分区类型](https://en.wikipedia.org/wiki/GUID_Partition_Table#Partition_type_GUIDs) | 建议大小     |
| ------------------------- | ------------- | ------------------------------------------------------------ | ------------ |
| `/mnt/boot` 或 `/mnt/efi` | `/dev/sd*X*1` | [EFI 系统分区](https://wiki.archlinux.org/index.php/EFI_system_partition_(简体中文)) | 260–512 MiB  |
| `/mnt`                    | `/dev/sd*X*2` | Linux x86-64 根目录 (/)                                      | 剩余空间     |
| `[SWAP]`                  | `/dev/sd*X*3` | Linux swap (交换空间)                                        | 大于 512 MiB |



使用 `fdisk` 建立分区：



- 新建分区表

```
fdisk /dev/sdX
```

1. 输入 `o`，新建 DOS 分区表
2. 输入 `w`，保存修改，这个操作会抹掉磁盘所有数据，慎重



- 分区创建

1 扇区 = 512 字节

```
fdisk /dev/sdX
```

1. 新建 Linux root 分区

1. 1. 输入 `n`
   2. 选择分区类型（p：主分区，e：扩展分区），默认选择 p ，直接 `Enter`
   3. 选择分区区号，直接 `Enter`，使用默认值，fdisk 会自动递增分区号
   4. 分区开始扇区号，直接 `Enter`，使用默认值
   5. 分区结束扇区号，这里要考虑预留给 swap 分区空间，计算公式：root 分区结束扇区号 = 磁盘结束扇区号 - 分配给 swap 分区的空间 (GB) * 1024 * 1024 * 1024 / 512，输入后 `Enter`
   6. 输入 `t` 修改刚刚创建的分区类型
   7. 选择分区号，直接 `Enter`， 使用默认值，fdisk 会自动选择刚刚新建的分区
   8. 输入 `83`，使用 Linux 类型

1. 新建 Linux swap 分区

1. 1. 输入 `n`
   2. 选择分区类型（p：主分区，e：扩展分区），默认选择 p ，直接 `Enter`
   3. 选择分区区号，直接 `Enter`，使用默认值，fdisk 会自动递增分区号
   4. 分区开始扇区号，直接 `Enter`，使用默认值
   5. 分区结束扇区号，直接 `Enter`，使用默认值
   6. 输入 `t` 修改刚刚创建的分区类型
   7. 选择分区号，直接 `Enter`， 使用默认值，fdisk 会自动选择刚刚新建的分区
   8. 输入 `82`，使用 Linux swap 类型

1. 保存新建的分区

1. 1. 输入 `w`



注：（1） `sdX` 是要分盘的盘名

​      （2）分盘的步骤只做参考即可你



### 格式化分区

当分区建立好了，这些分区都需要使用适当的 [文件系统](https://wiki.archlinux.org/index.php/File_systems_(简体中文)) 进行格式化。举个例子，如果根分区在 `/dev/sd*X*1` 上并且要使用 Ext4 文件系统，运行：

```
mkfs.ext4 /dev/sdX1
```

如果创建了 [交换分区](https://wiki.archlinux.org/index.php/Swap_(简体中文)) (例如 `/dev/*sda3*`)，请使用 [mkswap(8)](https://jlk.fjfi.cvut.cz/arch/manpages/man/mkswap.8) 将其初始化：

```
mkswap /dev/sdX2
swapon /dev/sdX2
```

详情请参阅 [文件系统](https://wiki.archlinux.org/index.php/File_systems_(简体中文)#创建文件系统)。



### 挂载分区

将根分区[挂载](https://wiki.archlinux.org/index.php/Mount)到 `/mnt`，例如：

```
mount /dev/sdX1 /mnt
```



## 安装

### 选择镜像

文件 `/etc/pacman.d/mirrorlist` 定义了软件包会从哪个[镜像源](https://wiki.archlinux.org/index.php/Mirrors)下载。在 LiveCD 启动的系统上，在连接到因特网后，[reflector](https://wiki.archlinux.org/index.php/Reflector) 会通过选择最近一个小时已同步的 HTTPS 镜像并按下载速率对其进行排序来更新镜像列表。[[1\]](https://gitlab.archlinux.org/archlinux/archiso/-/blob/master/configs/releng/airootfs/etc/systemd/system/reflector.service)

在列表中越前的镜像在下载软件包时有越高的优先权。您或许想检查一下文件，看看是否满意。如果不满意，可以相应的修改 `/etc/pacman.d/mirrorlist` 文件，并将地理位置最近的镜像源挪到文件的头部，同时也应该考虑一些其他标准。

这个文件接下来还会被 *pacstrap* 拷贝到新系统里，所以请确保设置正确。

```
vim /etc/pacman.conf
```

可以选择去掉 `#color` 的注释，无关大局。

```
vim /etc/pacman.d/mirrorlist
```

把中国的 Arch 源放到最前面。

也可以手动在该文件添加源地址，使用 Arch 提供的镜像生成器找到最新的镜像：[Pacman Mirrorlist Generator](https://www.archlinux.org/mirrorlist/)

```
##
## Arch Linux repository mirrorlist
## Generated on 2020-11-25
##

## China
#Server = http://mirrors.163.com/archlinux/$repo/os/$arch
#Server = http://mirrors.bfsu.edu.cn/archlinux/$repo/os/$arch
#Server = https://mirrors.bfsu.edu.cn/archlinux/$repo/os/$arch
#Server = http://mirrors.cqu.edu.cn/archlinux/$repo/os/$arch
#Server = https://mirrors.cqu.edu.cn/archlinux/$repo/os/$arch
#Server = http://mirrors.dgut.edu.cn/archlinux/$repo/os/$arch
#Server = https://mirrors.dgut.edu.cn/archlinux/$repo/os/$arch
#Server = http://mirrors.hit.edu.cn/archlinux/$repo/os/$arch
#Server = https://mirrors.hit.edu.cn/archlinux/$repo/os/$arch
#Server = http://mirror.lzu.edu.cn/archlinux/$repo/os/$arch
#Server = http://mirrors.neusoft.edu.cn/archlinux/$repo/os/$arch
#Server = https://mirrors.neusoft.edu.cn/archlinux/$repo/os/$arch
#Server = http://mirrors.nju.edu.cn/archlinux/$repo/os/$arch
#Server = https://mirrors.nju.edu.cn/archlinux/$repo/os/$arch
#Server = http://mirror.redrock.team/archlinux/$repo/os/$arch
#Server = https://mirror.redrock.team/archlinux/$repo/os/$arch
#Server = https://mirrors.sjtug.sjtu.edu.cn/archlinux/$repo/os/$arch
#Server = http://mirrors.tuna.tsinghua.edu.cn/archlinux/$repo/os/$arch
#Server = https://mirrors.tuna.tsinghua.edu.cn/archlinux/$repo/os/$arch
#Server = http://mirrors.ustc.edu.cn/archlinux/$repo/os/$arch
#Server = https://mirrors.ustc.edu.cn/archlinux/$repo/os/$arch
#Server = https://mirrors.xjtu.edu.cn/archlinux/$repo/os/$arch
#Server = http://mirrors.zju.edu.cn/archlinux/$repo/os/$arch
```

#### 【可选】添加 ArchLinuxCN 源



> ArchLinuxCN 是 Arch 中文组维护的一个软件合集，包含了中文用户常用的 WPS Office、搜狗拼音、Google Chrome 等软件。不过，Arch 本身和它的常见衍生版（如：Manjaro）默认都不包含这个源，因此我们需要手动来配置使用这个源。

如上所说，如果没装这些软件的需求可以先不配置。



**添加**

**
**

ArchLinuxCN 的源可以在这里[查看：[arch-linux-mirrorlist](https://github.com/archlinuxcn/mirrorlist-repo)



打开 `etc/pacman.conf` 。

```
# 打开 etc/pacman.conf
sudo code /etc/pacman.conf
```

添加你找到的镜像。

```
[archlinuxcn]
SigLevel = Optional TrustedOnly
Server = http://mirrors.tuna.tsinghua.edu.cn/archlinuxcn//$arch
```



**导入 GPG key 并更新系统**



看包名就能猜到这是 archlinuxcn 的 key，安装后作为验证，就可以安装软件了。

```
# 更新数据源
pacman -Syy
# 安装导入 GPG key
pacman -S archlinuxcn-keyring
# 更新系统
pacman -Syu
```

**注**：如果在安装 `sudo pacman -S archlinuxcn-keyring`  出现 “错误：未找到目标：archlinuxcn-keyring”，可以采用 `sudo pacman -S archlinux-keyring` 这个命令。

ss

### 安装必须的软件包

使用 [pacstrap](https://git.archlinux.org/arch-install-scripts.git/tree/pacstrap.in) 脚本，安装 [base](https://www.archlinux.org/packages/?name=base) 软件包和 Linux [内核](https://wiki.archlinux.org/index.php/Kernel)以及常规硬件的固件：

```
pacstrap /mnt base base-devel linux linux-firmware
```

提示：

- 可以将 [linux](https://www.archlinux.org/packages/?name=linux) 替换为 [kernel](https://wiki.archlinux.org/index.php/Kernel) 页面中介绍的内核软件包。
- 在虚拟机或容器中安装时，可以不安装固件软件包。

[base](https://www.archlinux.org/packages/?name=base) 软件包并没有包含 Live 环境中的全部程序。因此要获得一个功能齐全的基本系统，可能需要安装其他软件包。特别要考虑安装：

- 管理所用[文件系统](https://wiki.archlinux.org/index.php/File_systems)的用户工具；
- 访问 [RAID](https://wiki.archlinux.org/index.php/RAID) 或 [LVM](https://wiki.archlinux.org/index.php/LVM) 分区的工具；
- 未包含在 [linux-firmware](https://www.archlinux.org/packages/?name=linux-firmware) 中的额外固件；
- [联网](https://wiki.archlinux.org/index.php/Networking) 所需要的程序；
- [文本编辑器](https://wiki.archlinux.org/index.php/Text_editor)；
- 访问 [man](https://wiki.archlinux.org/index.php/Man) 和 [info](https://wiki.archlinux.org/index.php/Info) 页面的工具：[man-db](https://www.archlinux.org/packages/?name=man-db), [man-pages](https://www.archlinux.org/packages/?name=man-pages) 和 [texinfo](https://www.archlinux.org/packages/?name=texinfo)。

要 [安装](https://wiki.archlinux.org/index.php/Install) 其他软件包或软件包组 (比如 [base-devel](https://www.archlinux.org/groups/x86_64/base-devel/))，请将它们的名字追加到上文的 *pacstrap* 命令后 (用空格分隔)，或者也可以在 [Chroot 进新系统](https://wiki.archlinux.org/index.php/Installation_guide_(简体中文)#Chroot)后使用 [pacman](https://wiki.archlinux.org/index.php/Pacman) 手动安装软件包或软件包组。[packages.x86_64](https://gitlab.archlinux.org/archlinux/archiso/-/blob/master/configs/releng/packages.x86_64) 中可以看到不同软件包或软件包组间的差异。



## 配置系统

### Fstab

用以下命令生成 [fstab](https://wiki.archlinux.org/index.php/Fstab) 文件 (用 `-U` 或 `-L` 选项设置UUID 或卷标)：

```
genfstab -U /mnt >> /mnt/etc/fstab
```

**强烈建议**在执行完以上命令后，后检查一下生成的 `/mnt/etc/fstab` 文件是否正确。

### Chroot

[Change root](https://wiki.archlinux.org/index.php/Change_root_(简体中文)) 到新安装的系统：

```
arch-chroot /mnt
```

### 时区

设置[时区](https://wiki.archlinux.org/index.php/Time_zone)：

```
ln -sf /usr/share/zoneinfo/Region/City /etc/localtime
```

例如：

```
ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
```

运行 [hwclock(8)](https://jlk.fjfi.cvut.cz/arch/manpages/man/hwclock.8) 以生成 `/etc/adjtime`：

```
hwclock --systohc
```

这个命令假定硬件时间已经被设置为 [UTC 时间](https://en.wikipedia.org/wiki/UTC)。详细信息请查看 [System time#Time standard](https://wiki.archlinux.org/index.php/System_time#Time_standard)。



### 本地化

本地化的程序与库若要本地化文本，都依赖 [Locale](https://wiki.archlinux.org/index.php/Locale)，后者明确规定地域、货币、时区日期的格式、字符排列方式和其他本地化标准。

需在这两个文件设置：`locale.gen` 与 `locale.conf`。

编辑`/etc/locale.gen` 然后移除需要的 [地区](https://wiki.archlinux.org/index.php/Locale_(简体中文)) 前的注释符号 `#`。

接着执行 `locale-gen` 以生成 locale 信息。



步骤：

- 安装编辑器 vim，修改配置要使用

```
pacman -S gvim
```

- 修改 /etc/locale.gen，取消注释下面这两行配置

```
en_US.UTF-8 UTF-8
zh_CN.UTF-8 UTF-8
```

- 生成 locale 信息

```
locale-gen
```

- 然后创建 [locale.conf(5)](https://jlk.fjfi.cvut.cz/arch/manpages/man/locale.conf.5) 文件，并 [编辑设定 LANG 变量](https://wiki.archlinux.org/index.php/Locale#Setting_the_system_locale)，比如：

```
LANG=en_US.UTF-8
```



### 网络配置

创建 [hostname](https://wiki.archlinux.org/index.php/Hostname) 文件:

```
vim /etc/hostname

myhostname # 写入
```

添加对应的信息到 [hosts(5)](https://jlk.fjfi.cvut.cz/arch/manpages/man/hosts.5):

```
vim /etc/hosts

127.0.0.1   localhost
::1  localhost
127.0.1.1   myhostname.localdomain  myhostname
```

如果系统有一个永久的 IP 地址，请使用这个永久的 IP 地址而不是 `127.0.1.1`。

对新安装的系统，需要再次[设置网络](https://wiki.archlinux.org/index.php/Network_configuration_(简体中文))，请注意，目前的 [base](https://www.archlinux.org/packages/?name=base) 不含有任何网络管理工具，要安装希望使用的 [网络管理](https://wiki.archlinux.org/index.php/Network_management) 软件。



启动 dhcpcd 服务

```
pacman -S iw wpa_supplicant dialog net-tools networkmanager dhcpcd
systemctl enable dhcpcd
systemctl enable NetworkManager //大小写要注意
```



### Root 密码

设置 Root [密码](https://wiki.archlinux.org/index.php/Password)：

```
passwd
```



### 安装引导程序

需要安装 Linux 引导加载程序,才能在安装后启动系统，可以使用的的引导程序已在 [启动加载器](https://wiki.archlinux.org/index.php/Boot_loaders_(简体中文)) 中列出，请选择一个安装并配置它，[GRUB (简体中文)](https://wiki.archlinux.org/index.php/GRUB_(简体中文)) 是最常见的选择。

```
pacman -S grub
grub-install --target=i386-pc /dev/sdX
grub-mkconfig -o /boot/grub/grub.cfg
```

其中 `/dev/sd*X*` 是要安装 GRUB 的磁盘，比如磁盘 `/dev/sda`，而 **不是** 分区 `/dev/sda1`。



如果有 Intel 或 AMD 的 CPU，请另外启用 [微码](https://wiki.archlinux.org/index.php/Microcode_(简体中文)) 更新。



- AMD CPU

```
pacman -S amd-ucode
```

- Intel CPU

```
pacman -S intel-ucode
```



## 重启

输入 `exit` 或按 `Ctrl+d` 退出 chroot 环境。

可选用 `umount -R /mnt` 手动卸载被挂载的分区：这有助于发现任何「繁忙」的分区，并通过 [fuser(1)](https://jlk.fjfi.cvut.cz/arch/manpages/man/fuser.1) 查找原因。

最后，通过执行 `reboot` 重启系统，*systemd* 将自动卸载仍然挂载的任何分区。不要忘记移除安装介质，然后使用 root 帐户登录到新系统。



## 启动后需要设置的

- 开启时间自动同步

```
timedatectl set-ntp true
```

- 安装配置 openssl

```
pacman -S openssl
systemctl start sshd
systemctl enable sshd
```

- 配置 X11 转发

```
pacman -S xorg-xauth
# /etc/ssh/sshd_config

X11Forwarding yes
```

- 新建用户

```
useradd -m -G wheel rancbos
passwd rancbos # 为新用户设置密码
```

选项含义：

-m：创建用户目录，本例中会创建 /home/rancbos 文件夹

-G：设置用户的附属组，也就是将用户添加到其他组，但这些组是附属组，如果不添加用户到相应的附属组，则用户没有附属组相应的权限

wheel 组是一个很特殊的用户组，他被一些 Unix 系统用来控制能否通过 `su` 命令来切换到 root 用户。



Visudo 设置

```
visudo
```

进入后，找到

```
%whee ALL=(ALL) ALL # 然后删掉首位#，接着保存退出
```

删除用户账号

```
userdel -r rancbos
```

- 一些常用的软件

```
pacman -S zsh git tmux python python-pip xsel wget nodejs npm clang 
pacman -S ripgrep man-db man-pages texinfo cmake screenfetch boost gdb
pip install pynvim
pip install cpplint

pacman -S alsa-utils //声卡
pacman -S xf86-video-vesa //英特尔集显 安装这个驱动
pacman -S nvidia nvidia-settings //英伟达独显 安装这个驱动
```

- 蓝牙驱动

```
pacman -S bluez
systemctl enable bluetooth.service
```





### 安装显卡驱动、Vulkan 和硬件加速支持(可选)

Vulkan 是跨平台图形接口，目前主要用途是在 Linux 下运行 Windows 游戏以及部分 Linux 游戏（详见：[Linux 运行 Windows 游戏指南](https://www.mivm.cn/linux-run-windows-game/)），但并不是所有显卡都支持它，你可以前往 http://vulkan.gpuinfo.org/listreports.php?platform=linux 查询你的显卡是否支持 Vulkan。



#### AMD

显卡驱动（GCN 1 及以上架构）：`sudo pacman -S mesa lib32-mesa xf86-video-amdgpu libva-mesa-driver lib32-libva-mesa-driver mesa-vdpau lib32-mesa-vdpau opencl-mesa lib32-opencl-mesa ocl-icd`



显卡驱动（TeraScale 1 2 3 架构）：`sudo pacman -S mesa lib32-mesa xf86-video-ati libva-mesa-driver lib32-libva-mesa-driver mesa-vdpau lib32-mesa-vdpau opencl-mesa lib32-opencl-mesa ocl-icd`



Vulkan：`sudo pacman -S vulkan-icd-loader lib32-vulkan-icd-loader vulkan-radeon lib32-vulkan-radeon`



[andvlk](https://github.com/GPUOpen-Drivers/AMDVLK) 是由 AMD 开源的 AMD GPU Vulkan 实现，这个实现比 Mesa 的 RADV 在某些场景下性能和兼容性要好一些，并且系统允许安装多个 Vulkan 实现并让软件自行选择，推荐游戏玩家安装。



#### Intel

显卡驱动：`sudo pacman -S mesa lib32-mesa xf86-video-intel`



硬件视频加速：如果你的 Intel CPU 架构是 Broadwell (6代) 或以上的安装`intel-media-driver`，反之则安装`libva-intel-driver lib32-libva-intel-driver`。



OpenCL：如果你的 Intel CPU 架构是 Broadwell (6代) 或以上的安装`intel-compute-runtime ocl-icd`，反之则安装`beignet ocl-icd`，也可以选择不安装。



Vulkan：`sudo pacman -S vulkan-icd-loader lib32-vulkan-icd-loader vulkan-intel lib32-vulkan-intel`



#### Nvidia

GeForce 630 及以上型号：`sudo pacman -S nvidia nvidia-settings xorg-server-devel lib32-nvidia-utils opencl-nvidia lib32-opencl-nvidia ocl-icd cuda`



GeForce 620 及以下型号（开源驱动）：`sudo pacman -S mesa lib32-mesa xf86-video-nouveau libva-mesa-driver lib32-libva-mesa-driver mesa-vdpau lib32-mesa-vdpau`



由于 GT620 及以下型号不被 Nvidia 最新官方驱动支持，推荐使用开源驱动。



**PS：如果是双显卡的笔记本，运行软件时可以选择使用的显卡（默认是集成显卡）。参见：https://wiki.archlinux.org/index.php/PRIME**









## 小技巧：

### 网络连接

第一次安装的时候忘了安装网络工具，有没有网线，于是上网查找方法，发现了用手机连接 wifi 的方式——手机网络共享。感觉非常有用，颇为惊喜。

手机连接 wifi(当然流量也行) 以后，用数据线连到电脑，开启 "usb网络共享" 功能，最后 `dhcpcd ` 即可。







## 参考文档

1、ArchLinux Wiki ：[Installation guide](https://wiki.archlinux.org/index.php/Installation_guide_(简体中文))

2、[Arch Linux (BIOS with MBR) 安装](https://shenyu.me/2020/04/10/arch-bios-install.html)