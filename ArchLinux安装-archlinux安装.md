---
title: ArchLinux安装
date: 2021-09-05 12:15:04.258
updated: 2021-09-09 00:26:38.239
url: /archives/archlinux安装
categories: 系统
tags: linux
---

#### 1.制作系统U盘


#### 2.开机U盘启动

#### 3.配置系统

1. 查看系统分区，找到需要安装系统的磁盘设备（此处选择 /dev/sda）
```bash
$ lsblk
sda      8:0    0 111.8G  0 disk
sdb      8:16   0 931.5G  0 disk /data
```
2. 新建分区，挂载分区
```bash
查看磁盘设备分区:
$ fdisk /dev/sda
    n:新建分区 
    t:格式化分区类型
    w:保存
    p:打印当前分区

分区类型示例:
# boot    600M    ef00    mkfs.fat -F32
# swap    4G      8200    mkswap , swapon
# system  32G     8300    mkfs.ext4      作为跟目录
# home    remain  8300    mkfs.ext4	

分区结果:
$ lsblk
sda      8:0    0 111.8G  0 disk
├─sda1   8:1    0   600M  0 part 
├─sda2   8:2    0     4G  0 part 
├─sda3   8:3    0    32G  0 part 
└─sda4   8:4    0  75.2G  0 part 

挂载分区:
$ mkdir /mnt/boot /mnt/home
$ mount /dev/sda3 /mnt
$ mount /dev/sda1 /mnt/boot
$ mount /dev/sda4 /mnt/home

配置磁盘自动挂载:
$ genfstab -U /mnt >> /mnt/etc/fstab

```
3. 更新系统时间
```bash
$ timedatectl set-ntp true
```

4. 更新镜像源
```bash
获取中国镜像源地址数据:
$ curl 'https://www.archlinux.org/mirrorlist/?country=CN&protocol=http&protocol=https&ip_version=4&use_mirror_status=on' > /etc/pacman.d/mirrorlist

开启服务地址:
$ sed -i "s/#Server/Server/g"  /etc/pacman.d/mirrorlist
```

5. 安装基础系统，软件
```bash
$ pacstrap /mnt base linux linux-firmware man-db man-pages base-devel vim
```


6. 进入新系统
```bash
$ arch-chroot /mnt
```

7. 配置时区
```bash
$ ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
$ hwclock --systohc
```

8. 本地化配置
```bash
开启en_US.UTF-8, 并生成配置
$ vim /etc/locale.gen
$ locale-gen

创建locale.conf
$ touch /etc/locale.conf
$ echo 'LANG=en_US.UTF-8' >> /etc/locale.conf
```

9. 配置网络
```bash
设置主机名:
$ echo 'archlinux' > /etc/hostname

配置hosts
$ echo '\
    127.0.0.1   localhost\
    ::1     localhost\
    127.0.1.1   archlinux.localdomain   archlinux' >> /etc/hosts
```

10. 安装引导程序GRUB
```bash
安装GRUB
$ pacman -S --noconfirm grub efibootmgr

配置GRUB
$ grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=GRUB
$ grub-mkconfig -o /boot/grub/grub.cfg
```

11. 修改ROOT密码
```bash
$ passwd root
```

12. 退出新系统
```bash
$ exit
```

13. 重启系统
```bash
卸载挂载的分区
$ umount -R /mnt

重启系统
$ reboot now
```


*以上部分已经配置完毕，重启后即可登陆ArchLinux系统*

*下面是配置系统网卡，桌面环境*


1. 配置网卡
```bash
查看网卡信息(此处是 enp0s31f6):
$ ip link list
1: lo: <LOOPBACK,UP,LOWER_UP> 
2: enp0s31f6: <BROADCAST,MULTICAST,UP,LOWER_UP> 

如果网卡未开启则需要手动开启:
$ ip link set enp0s31f6 up

创建网卡配置文件:
$ touch /etc/systemd/network/enp0s31f6.network

编辑网卡配置文件:
$ vim /etc/systemd/network/enp0s31f6.network
内容如下: 
[Match]
Name=enp0s31f6
[Network]
Address=192.168.1.200/24
Gateway=192.168.1.1
DNS=8.8.8.8

启动网络服务:
$ systemctl start systemd-networkd.service
$ systemctl start systemd-resolved.service

配置网络服务开机自启:
$ systemctl enable systemd-networkd.service
$ systemctl enable systemd-resolved.service
```

2. 更新软件源
```bash
$ pacman -Syyu --noconfirm
```

3. 安装软件
```bash
$ pacman -S --noconfirm openssh
$ pacman -S --noconfirm zsh zsh-completions
```


4. 创建普通用户
```bash
创建普通用户，并指定shell为zsh:
$ useradd -s /bin/zsh -m docryze

修改密码:
$ passwd docryze

```


5. 安装桌面环境(可以选择dwm或者i3)

- dwm桌面环境
```bash
安装必要的窗口环境：
$ pacman -S org-server org-xinit org-xrandr xorg-xsetroot libxft libxinerama picom 

如果是nvidia显卡,可以安装驱动,自动生成配置/etc/X11/xorg.conf
$ pacman -S nvidia
$ nvidia-xconfig

安装dwm
$ git clone https://git.suckless.org/dwm
$ sudo make clean install

配置/usr/share/xsessions:
$ touch /usr/share/xsessions/dwm.desktop
$ vim /usr/share/xsessions/dwm.desktop
输入以下内容
[Desktop Entry]
Name=dwm
Comment=dynamic window manager
Exec=dwm
TryExec=dwm
Type=Application
Icon=/home/docryze/soft/dwm/dwm.png

patch < autostart.diff
touch ~/.dwm/autostart.sh
chmod +x ~/.dwm/autostart.sh
```

- i3桌面环境
```bash
$ sudo pacman -S i3
```

- 安装字体
```bash
pacman -S adobe-source-han-serif-cn-fonts 
pacman -S adobe-source-code-pro-fonts
```


6. 安装登陆管理器,配置自启动

```bash
$ pacman -S lightdm lightdm-gtk-greeter
$ systemctl enable ligthdm.service
```

7. 重启系统，用刚才创建的普通用户登陆


#### 4.软件推荐
```bash
zsh管理软件:
	1.oh-my-zsh
vim插件管理:
	1.vim-plug
搜索工具:
	1.fzf
终端软件:
	1.st
文件管理器:
	1.ranger
	源码安装：
	    sudo paceman -Ss python3 python-setuputils 
	    git clone url
	    sudo make clean install
渲染优化:
	1.picom
	$ pacman -S picom
搜索工具:
	1.albert
	$ pacman -S albert

	2.i3自带的dmenu
壁纸:
	1.feh
	pacman -S feh
声音配置:
	1.alsa-utils
	$ pacman -S alsa-utils #使用alsamixer在控制台配置声音
```







