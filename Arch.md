-------------------------------------------------------------------------------
# 安装 Arch Linux

## 0. U盘安装

遇到找不到iso，等进入 `[bootfs ]` 后，`ls /dev/disk/by-label/` 查看当前 `label`
有两种方法：

### 0.1

```
$ mv /dev/disk/by-label/YOUR_LABEL /dev/disk/by-label/ARCH_XXXX  # XXXX 为引导界面的配置(ARCH_XXXX)
```

### 0.2

进入引导按 `TAB` 或者 `E` 编辑引导参数，将 `ARCH_XXXX` 改为 **YOUR_LABEL**

## 1. 测试网络状态（插着网线，WiFi没有尝试过）

```sh
# ping -c 2 www.baidu.com
# systemctl enable dhcpcd.service               # 若ping失败
```

## 2. 测试系统时间

```sh
# timedatectl status
# timedatectl set-ntp true                      # 若时间不对
```

## 3. 磁盘分区&挂载目录

```sh
# lsblk                                         # or fdisk -l
# fdisk /dev/sdaX                               # n, <Enter>, <Enter>, w
# mkfs.ext4 /dev/sda1
# mount /dev/sda1 /mnt
```

## 4. 修改软件源镜像

```sh
# cd /etc/pacman.d
# grep -A 1 'China' mirrorlist | grep -v '\-\-' > mr
# cat mirrorlist >> mr && mv mr mirrorlist
```

## 5. 安装基本系统

```sh
# pacstrap -i /mnt base base-devel
```

## 6. 生成fstab

```sh
# genfstab -U /mnt >> /mnt/etc/fstab
# cat /mnt/etc/fstab                            # 查看一下是否生成成功
```

## 7. 进入到新系统 & 基本配置

```sh
# arch-chroot /mnt /bin/bash
# vi /etc/locale.gen                            # 打开 en_US.UTF-8, zh_CN.UTF-8
# locale-gen                                    # 生成区域
# echo LANG=en_US.UTF-8 > /etc/locale.conf      # 不安装桌面

# ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime   # 配置时区
# echo ArchLinux > /etc/hostname
# vi /etc/hosts

# passwd                                        # 设置root密码
# pacman -S grub                                # 安装引导工具
# grub-install --recheck /dev/sda
# grub-mkconfig -o /boot/grub/grub.cfg          # 默认配置

# systemctl enable dhcpcd.service               # 配置网络
# exit
# umount -R /mnt
# reboot
```

## 8. 双系统grub设置（Windows 10 & Arch Linux）

当分不清是使用 *grub-legacy* 还是 *efi* 时，用以下方法查找 **Windows 10** 安装在何处，并逐个进入查看，通常默认安装在(hd0,1)

```sh
# vi /etc/grub.d/40_custom                      # 在文件中增加如下内容
```

    menuentry "Microsoft Windows 10 - 1" {      # 通常默认就是这个
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
# grub-mkconfig -o /boot/grub/grub.cfg     # 重新生成grub引导
# reboot
```

参考资料: [Dual-boot archlinux and Windows 7 using grub-bios](https://superuser.com/questions/528975/dual-boot-archlinux-and-windows-7-using-grub-bios)

-------------------------------------------------------------------------------

-------------------------------------------------------------------------------
# 系统配置

## 1. 添加用户

```sh
# useradd -m fanbin
# passwd
# visudo                                        # 增加权限
# exit                                          # 退出并切换用户
```

## 2. 装 X

```sh
$ sudo pacman -S xorg xorg-xinit                # xorg-xeyes xorg-xclock（可不装）
$ sudo pacman -S xterm
```

此时startx就能看到 win界面

## 3. 装yaourt

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

## 4. 基本软件（根据个人爱好安装）

### 4.1 工具

```sh
$ sudo pacman -S gcc gdb                            # gcc gdb
$ sudo pacman -S cgdb                               # cgdb
$ sudo pacman -S vim
$ sudo pacman -S ctags                              # 查找定义
$ sudo pacman -S cscope                             # 查找静态调用
$ sudo pacman -S automake autoconf m4 perl libtool  # make 套件
$ sudo pacman -S git svn
$ sudo pacman -S wget curl
$ sudo pacman -S zip unzip
$ sudo pacman -S acpi                               # 笔记本电池信息查看
```

### 4.2 开发环境&语言

```sh
$ sudo pacman -S python lua
$ sudo pacman nodejs npm yarn
```

#### 4.2.1 Golang

```sh
$ sudo pacman -S go godef gocode
$ vim ~/.xinitrc

    export GOPATH=~/workspace/go/
    export GOROOT=/usr/lib/go/
    export PATH=$PATH:$GOPATH/bin

$ go get -u PACKAGE
$ go install PACKAGE
```

#### 4.2.2 php + php-fpm + nginx

```sh
$ sudo pacman -S php
$ sudo pacman -S php-fpm
$ sudo pacman -S nginx

$ sudo systemctl start php-fpm
$ sudo systemctl start nginx
```

### 4.3 APP

```sh
$ sudo pacman -S firefox                        # Firefox（推荐）
$ sudo pacman -S thunderbird                    # 邮件客户端
$ sudo pacman -S evince                         # PDF viwer
$ yaourt google-chrome                          # Chrome
$ yaourt netease-musicbox-git                   # https://github.com/darknessomi/musicbox
$ yaourt wps-office                             # WPS OFFICE（办公套件）
$ yaourt youcompleteme                          # YouCompleteMe

$ sudo pacman -S virtualbox-host-modules-arch   # VirtualBox
$ sudo modprobe vboxdrv
$ sudo depmod -a

$ sudo pacman -S xrandr                         # 屏幕设置及扩展屏幕
$ sudo pacman -S arandr                         # xrandr GUI

$ yaourt gnome-alsamixer                        # 声音调节 GUI
$ sudo pacman -S xf86-input-synaptics           # 笔记本触摸板驱动
```

参考资料: 
* [VirtualBox - Arch Wiki](https://wiki.archlinux.org/index.php/VirtualBox)
* [Xrandr - Arch Wiki](https://wiki.archlinux.org/index.php/Xrandr)
* [Touchpad Synaptics](https://wiki.archlinux.org/index.php/Touchpad_Synaptics)

### 4.4 神器

```sh
$ sudo pacman -S tmux
$ sudo pacman -S tig                 # https://github.com/jonas/tig
$ sudo pacman -S the_silver_searcher # https://github.com/ggreer/the_silver_searcher
$ sudo pacman -S fzf                 # https://github.com/junegunn/fzf (git install)
$ sudo pacman -S htop                # https://hisham.hm/htop/
$ sudo pacman -S axel                # http://axel.alioth.debian.org/
$ sudo pacman -S cloc                # http://cloc.sourceforge.net/
$ sudo pacman -S thefuck             # https://github.com/nvbn/thefuck
$ sudo pacman -S httpie              # https://httpie.org/
$ sudo pacman -S jq                  # https://stedolan.github.io/jq/
$ sudo pacman -S tree
$ yaourt tldr                        # https://github.com/tldr-pages/tldr
$ yaourt mycli                       # https://github.com/dbcli/mycli（安装过程比较慢）

$ # https://github.com/Xfennec/progress
$ git clone https://github.com/Xfennec/progress.git; cd progress; make && sudo make install
```

## 5. i3 + st + conky

先使用这个配置 [ivyl/i3-config](https://github.com/ivyl/i3-config)

```sh
$ sudo pacman -S i3                  # i3 window manager(i3-wm, i3blocks, i3lock, i3status)
$ sudo pacman -S dmenu               # Launcher
$ sudo pacman -S conky               # monitor
$ sudo pacman -S udiskie             # device automounting
$ sudo pacman -S autocutsel          # clipboard synchroniation
$ yaourt i3lock-fancy-git            # 更美观的锁屏

    git clone https://github.com/meskarune/i3lock-fancy.git
    cd i3lock-fancy && sudo cp -r lock icons/ /usr/local/bin
    cd ../ && rm -rf i3lock-fancy-git

```

用我的配置替换 *ivyl/i3-config* 的配置(TODO)

参考资料: 
* [i3](https://i3wm.org/) - improved tiling wm
* [i3-gaps](https://github.com/Airblader/i3) - i3 with more features
* [i3lock-fancy](https://github.com/meskarune/i3lock-fancy) - blurs the background and adds a lock icon and text
* [st](http://st.suckless.org/) - st is simple terminal implementation for X
* [conky](https://github.com/brndnmtthws/conky) - light-weight system monitor for X
* [dmenu](http://tools.suckless.org/dmenu/) - dmenu is a dynamic menu for X

*i3-wm* 透明：

```sh
$ sudo pacman -S xcompmgr
$ sudo pacman -S transset-df        # 使用transset-df设置透明度
$ xcompmgr -c
```

参考资料：
* [Per-application transparency](https://wiki.archlinux.org/index.php/Per-application_transparency)

## 6. [oh-my-zsh](https://github.com/robbyrussell/oh-my-zsh)

```sh
$ sudo pacman -S zsh
$ sh -c "$(wget https://raw.github.com/robbyrussell/oh-my-zsh/master/tools/install.sh -O -)"
```

## 7. [oh-my-tmux](https://github.com/gpakosz/.tmux)

```sh
$ sudo pacman -S tmux
$ cd
$ git clone https://github.com/gpakosz/.tmux.git
$ ln -s -f .tmux/.tmux.conf
$ cp .tmux/.tmux.conf.local .
```

## 8. 字体（自选）

```sh
$ git clone https://github.com/powerline/fonts.git
$ cd fonts && ./install.sh && cd .. && rm -rf fonts

$ sudo pacman -S ttf-dejavu                             # 安装字体(我的 `st` 用了此字体)
$ sudo pacman -S wqy-zenhei wqy-microhei                # 其他字体
$ fc-cache -vf                                          # 刷新字体缓存
$ fc-match                                              # 查看当前默认字体
```

参考资料: [Powerline](https://github.com/powerline/fonts) - patched fonts for Powerline users

## 9. 输入法

```sh
$ sudo pacman -S fcitx-im fcitx-configtool
$ sudo pacman -S fcitx-sunpinyin                        # 输入发
$ sudo pacman -S fcitx-cloudpinyin fcitx-sogoupinyin    # 其他输入法
$ fcitx                                                 # 打开输入法
```

右上角鼠标右键打开 **Input Method**，取消勾选 **Only Show Current Language**，搜索 **Sunpinyin**，添加输入法

参考资料：[fcitx - Arch Wiki](https://wiki.archlinux.org/index.php/Fcitx)

## 10. Vim

使用我的 *Vim* 配置

```sh
$ sudo pacman -S vim
$ git clone https://github.com/yuanfanbin/dotfile.git ~/.dotfile

$ ln -s ~/.dotfile/.vimrc ~/.vimrc
$ # 或者
$ cp ~/.dotfile/.vimrc ~/

$ mkdir -p ~/.vim/bundle
$ git clone https://github.com/VundleVim/Vundle.vim.git ~/.vim/bundle/Vundle.vim
$ vim +PluginInstall +qall
```

[VimScript脚本语言学习](http://blog.csdn.net/smstong/article/details/20730587)
[VimScript编程指南](https://kenvifire.gitbooks.io/vimscript/content/about.html)

### 10.1 dotfile(杂七杂八)

```sh
$ ln -s ~/.dotfile/.tigrc ~/.tigrc
$ ln -s ~/.dotfile/.ackrc ~/.ackrc
$ ln -s ~/.dotfile/.tmux.conf ~/.tmux.conf
```

## 11. Emacs([spaceemacs](https://github.com/syl20bnr/spacemacs))

TODO
-------------------------------------------------------------------------------
