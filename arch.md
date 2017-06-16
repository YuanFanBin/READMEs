-------------------------------------------------------------------------------
# 安装 Arch Linux

# 0. U盘安装

遇到找不到iso，等进入 `[bootfs ]` 后，`ls /dev/disk/by-label/` 查看当前 `label`
有两种方法：

## 1

```
$ mv /dev/disk/by-label/YOUR_LABEL /dev/disk/by-label/ARCH_XXXX  # XXXX 为引导界面的配置(ARCH_XXXX)
```

## 2
进入引导按 `TAB` 或者 `E` 编辑引导参数，将 `ARCH_XXXX` 改为 **YOUR_LABEL**

# 1. 测试网络状态

```sh
# ping -c 2 www.baidu.com
# systemctl enable dhcpcd.service   # 若ping失败
```

# 2. 测试系统时间

```sh
# timedatectl status
# timedatectl set-ntp true          # 若时间不对
```

# 3. 磁盘分区&挂载目录

```sh
# lsblk     # or fdisk -l
# fdisk /dev/sda    # n, <Enter>, <Enter>, w
# mkfs.ext4 /dev/sda1
# mount /dev/sda1 /mnt
```

# 4. 修改软件源镜像

```sh
# cd /etc/pacman.d
# grep -A 1 'China' mirrorlist | grep -v '\-\-' > mr
# cat mirrorlist >> mr && mv mr mirrorlist
```

# 5. 安装基本系统（网速慢的话就不要安装了）

```sh
# pacstrap -i /mnt base base-devel
```

# 6. 生成fstab

```sh
# genfstab -U /mnt >> /mnt/etc/fstab
# cat /mnt/etc/fatab                    # 查看一下是否生成成功
```

# 7. 进入到新系统 & 基本配置

```sh
# arch-chroot /mnt /bin/bash
# vi /etc/locale.gen            # 打开 en_US.UTF-8, zh_CN.UTF-8, zh_TW.UTF-8
# locale-gen                    # 生成区域
# echo LANG=en_US.UTF-8 > /etc/locale.conf  # 不安装桌面
# echo LANG=zh_CN.UTF-8 > /etc/locale.conf  # 安装桌面

# ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime   # 配置时区
# echo archlinux > /etc/hostname
# vi /etc/hosts

# passwd                                    # 设置root密码
# pacman -S grub                            # 安装引导工具
# grub-install --recheck /dev/sda
# grub-mkconfig -o /boot/grub/grub.cfg      # 默认配置

# systemctl enable dhcpcd.service           # 配置网络
# exit
# umount -R /mnt
# reboot
```

# 8. 双系统grub设置（Windows 10 & Arch Linux）

当分不清是使用 *grub-legacy* 还是 *efi* 时，用以下方法查找 **Windows 10** 安装在何处，并逐个进入查看，通常默认安装在(hd0,1)

```sh
$ sudo vi /etc/grub.d/40_custom                 # 在文件中增加如下内容
```

    menuentry "Microsoft Windows 10 - 1" {
        set root=(hd0,1)
        chainloader (hd0,1)+1
    }
    menuentry "Microsoft Windows 10 - 2" {
        set root=(hd0,2)
        chainloader (hd0,2)+1
    }
    menuentry "Microsoft Windows 10 - 3" {
        set root=(hd0,3)
        chainloader (hd0,3)+1
    }

```sh
$ sudo grub-mkconfig -o /boot/grub/grub.cfg     # 重新生成grub引导
$ sudo reboot
```

参考资料：https://superuser.com/questions/528975/dual-boot-archlinux-and-windows-7-using-grub-bios
-------------------------------------------------------------------------------

-------------------------------------------------------------------------------
# 系统配置

# 1. 添加用户

```sh
$ useradd -m fanbin
$ passwd fanbin
$ visudo                #   增加权限
```

# 2. 装 X

```sh
$ sudo pacman -S xorg xorg-xinit xterm xorg-xeyes xorg-xclock
```

此时startx就能看到 win界面

# 3. 装yaourt

```sh
$ sudo vi /etc/pacman.conf
```

    [archlinuxcn]
    SigLevel = Optional TrustedOnly
    Server = https://mirrors.tuna.tsinghua.edu.cn/archlinuxcn/$arch

```sh
$ sudo pacman -Sy archlinuxcn-keyring yaourt
```

这里添加的 ArchLinuxCN 源是 archlinuxcn.org 的第三方源，包括了不少常用软件。

# 4. 基本软件（根据个人爱好安装）

## 工具

```sh
$ sudo pacman -Sy gcc gdb                           # gcc gdb
$ sudo pacman -Sy cgdb                              # cgdb
$ sudo pacman -Sy vim ctags cscope
$ sudo pacman -Sy automake autoconf m4 perl libtool # make 套件
$ sudo pacman -Sy git svn
$ sudo pacman -S wget curl
```

## 语言

```sh
$ sudo pacman -Sy php python go lua
$ sudo pacman nodejs npm yarn                       # for web dev
```

## 软件

```sh
sudo pacman -Sy nginx php-fpm
```

## APP

```sh
$ yaourt google-chrome
```

## 神器

```sh
$ sudo pacman -S tmux
$ sudo pacman -S tig                 # https://github.com/jonas/tig
$ sudo pacman -S the_silver_searcher # https://github.com/ggreer/the_silver_searcher
$ yaourt mycli                       # https://github.com/dbcli/mycli
$ sudo pacman -S fzf                 # https://github.com/junegunn/fzf (git install)
$ sudo pacman -S htop                # https://hisham.hm/htop/
$ sudo pacman -S axel                # http://axel.alioth.debian.org/
$ sudo pacman -S cloc                # http://cloc.sourceforge.net/
$ sudo pacman -S thefuck             # https://github.com/nvbn/thefuck
$ yaourt tldr                        # https://github.com/tldr-pages/tldr
$ sudo pacman -S httpie              # https://httpie.org/
$ sudo pacman -S jq                  # https://stedolan.github.io/jq/
$ yaourt musicbox                    # https://github.com/darknessomi/musicbox
```

# 5. i3

先使用这个配置 https://github.com/ivyl/i3-config

```sh
$ sudo pacman -Sy i3 i3status feh
$ yaourt urxvt
$ sudo pacman -Sy unclutter udiskie dunst pulseaudio autocutsel dmenu pavucontrol
$ yaourt alsamixer
$ sudo pacman -S conky          # 这个软件作者没写，也需要安装(for i3bar, monitor)
$ git clone https://github.com/meskarune/i3lock-fancy.git # 这个是锁屏
```

# 6. Zsh

```sh
$ pacman -S zsh
sh -c "$(wget https://raw.github.com/robbyrussell/oh-my-zsh/master/tools/install.sh -O -)"
```

# 7. 字体

```sh
$ git clone https://github.com/powerline/fonts.git
$ cd fonts && ./install.sh && cd .. && rm -rf fonts

$ pacman -S wqy-microhei wqy-zenhei noto-fonts noto-fonts-cjk
```

# 8. 输入法

```sh
$ pacman -S fcitx-im fcitx-configtool
$ pacman -S fcitx-sunpinyin fcitx-cloudpinyin fcitx-sogoupinyin # 各种输入法
```

https://wiki.archlinux.org/index.php/Fcitx_(简体中文)
-------------------------------------------------------------------------------
