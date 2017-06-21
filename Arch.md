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

# 1. 测试网络状态（插着网线，WiFi没有尝试过）

```sh
# ping -c 2 www.baidu.com
# systemctl enable dhcpcd.service               # 若ping失败
```

# 2. 测试系统时间

```sh
# timedatectl status
# timedatectl set-ntp true                      # 若时间不对
```

# 3. 磁盘分区&挂载目录

```sh
# lsblk                                         # or fdisk -l
# fdisk /dev/sdaX                               # n, <Enter>, <Enter>, w
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
# cat /mnt/etc/fstab                            # 查看一下是否生成成功
```

# 7. 进入到新系统 & 基本配置

```sh
# arch-chroot /mnt /bin/bash
# vi /etc/locale.gen                            # 打开 en_US.UTF-8, zh_CN.UTF-8
# locale-gen                                    # 生成区域
# echo LANG=en_US.UTF-8 > /etc/locale.conf      # 不安装桌面
# echo LANG=zh_CN.UTF-8 > /etc/locale.conf      # 安装桌面

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

# 8. 双系统grub设置（Windows 10 & Arch Linux）

当分不清是使用 *grub-legacy* 还是 *efi* 时，用以下方法查找 **Windows 10** 安装在何处，并逐个进入查看，通常默认安装在(hd0,1)

```sh
$ sudo vi /etc/grub.d/40_custom                 # 在文件中增加如下内容
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
$ sudo grub-mkconfig -o /boot/grub/grub.cfg     # 重新生成grub引导
$ sudo reboot
```

参考资料: [Dual-boot archlinux and Windows 7 using grub-bios](https://superuser.com/questions/528975/dual-boot-archlinux-and-windows-7-using-grub-bios)
-------------------------------------------------------------------------------

-------------------------------------------------------------------------------
# 系统配置

# 1. 添加用户

```sh
$ useradd -m fanbin
$ passwd fanbin
$ visudo                                        # 增加权限
```

# 2. 装 X

```sh
$ sudo pacman -S xorg xorg-xinit                # xorg-xeyes xorg-xclock（可不装）
$ sudo pacman -S xterm
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
$ sudo pacman -S gcc gdb                        # gcc gdb
$ sudo pacman -S cgdb                           # cgdb
$ sudo pacman -S vim ctags cscope
$ sudo pacman -S automake autoconf m4 perl libtool # make 套件
$ sudo pacman -S git svn
$ sudo pacman -S wget curl
```

## 开发环境&语言

```sh
$ sudo pacman -S php python go lua
$ sudo pacman -S nginx php-fpm
$ sudo pacman nodejs npm yarn                   # for web dev
```

## APP

```sh
$ sudo pacman -S firefox                        # Firefox（推荐）
$ yaourt google-chrome                          # Chrome
$ sudo pacman -S thunderbird                    # 邮件客户端
$ sudo pacman -S virtualbox
```

## 神器

```sh
$ sudo pacman -S tmux
$ sudo pacman -S tig                 # https://github.com/jonas/tig
$ sudo pacman -S the_silver_searcher # https://github.com/ggreer/the_silver_searcher
$ yaourt mycli                       # https://github.com/dbcli/mycli（安装过程比较慢）
$ sudo pacman -S fzf                 # https://github.com/junegunn/fzf (git install)
$ sudo pacman -S htop                # https://hisham.hm/htop/
$ sudo pacman -S axel                # http://axel.alioth.debian.org/
$ sudo pacman -S cloc                # http://cloc.sourceforge.net/
$ sudo pacman -S thefuck             # https://github.com/nvbn/thefuck
$ yaourt tldr                        # https://github.com/tldr-pages/tldr
$ sudo pacman -S httpie              # https://httpie.org/
$ sudo pacman -S jq                  # https://stedolan.github.io/jq/
$ yaourt netease-musicbox-git        # https://github.com/darknessomi/musicbox
$ sudo pacman -S tree
```

# 5. i3 + st + conky

先使用这个配置 [ivyl/i3-config](https://github.com/ivyl/i3-config)

```sh
$ sudo pacman -S i3                  # i3 window manager(i3-wm, i3blocks, i3lock, i3status)
$ sudo pacman -S udiskie             # device automounting
$ sudo pacman -S autocutsel          # clipboard synchroniation
$ sudo pacman -S dmenu               # Launcher
$ sudo pacman -S conky               # monitor
$ yaourt i3lock-fancy-git            # 更美观的锁屏

    git clone https://github.com/meskarune/i3lock-fancy.git
    cd i3lock-fancy && sudo cp -r lock icons/ /usr/local/bin
    cd ../ && rm -rf i3lock-fancy-git

```

用我的配置替换 *ivyl/i3-config* 的配置

```sh
$ yaourt alsamixer              # 声音调节
```

# 6. [oh-my-zsh](https://github.com/robbyrussell/oh-my-zsh)

```sh
$ pacman -S zsh
$ sh -c "$(wget https://raw.github.com/robbyrussell/oh-my-zsh/master/tools/install.sh -O -)"
```

# 7. 字体（自选）

## [Powerline](https://github.com/powerline/fonts)

```sh
$ git clone https://github.com/powerline/fonts.git
$ cd fonts && ./install.sh && cd .. && rm -rf fonts

$ sudo pacman -S ttf-dejavu                             # 安装字体(我的 `st` 用了此字体)
$ sudo pacman -S wqy-zenhei wqy-microhei                # 其他字体
$ fc-cache -vf                                          # 刷新字体缓存
$ fc-match                                              # 查看当前默认字体
```

# 8. 输入法

```sh
$ sudo pacman -S fcitx-im fcitx-configtool
$ sudo pacman -S fcitx-sunpinyin                        # 输入发
$ sudo pacman -S fcitx-cloudpinyin fcitx-sogoupinyin    # 其他输入法
$ fcitx                                                 # 打开输入法
```

右上角鼠标右键打开 **Input Method**，取消勾选 **Only Show Current Language**，搜索 **Sunpinyin**，添加输入法

参考资料：[fcitx - Arch Wiki](https://wiki.archlinux.org/index.php/Fcitx)

# 9. Vim

使用我的 *Vim* 配置

```sh
$ sudo pacman -S vim
$ git clone https://github.com/yuanfanbin/dotfile.git ~/github/yuanfanbin/dotfile   # 若刚才没有clone过，则clone一次

$ ln -s ~/github/yuanfanbin/dotfile/.vimrc ~/.vimrc
$ # 或者
$ cp ~/github/yuanfnabin/dotfile/.vimrc ~/

$ mkdir -p ~/.vim/bundle
$ git clone https://github.com/VundleVim/Vundle.vim.git ~/.vim/bundle/Vundle.vim
$ vim +PluginInstall +qall

$ ln -s `pwd`/conf/.vimrc ~/.vimrc
$ mkdir -p ~/.vim/bundle
$ git clone https://github.com/VundleVim/Vundle.vim.git ~/.vim/bundle/Vundle.vim
```

# 9.1 dotfile(杂七杂八)

```sh
$ ln -s ~/github/yuanfnabin/dotfile/.tigrc ~/.tigrc
$ ln -s ~/github/yuanfnabin/dotfile/.ackrc ~/.ackrc
$ ln -s ~/github/yuanfnabin/dotfile/.tmux.conf ~/.tmux.conf
```

# 10. Emacs([spaceemacs](https://github.com/syl20bnr/spacemacs))

TODO
-------------------------------------------------------------------------------
