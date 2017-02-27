---
title: '使用LVM+Btrfs+EFI安装Arch Linux'
date: 2017-2-27 21:13:20
tags: Linux
---



一直感觉使用ubuntu这类开箱即用的linux系统很不灵活，每次安装完成都需要设置很多东西。安装ubuntu类似一个黑箱操作，很多东西都不透明。为了追求极致的简洁和自由，开始尝试研究安装Arch Linux,作为最优秀的元发行版。其实arch linux一直受到广泛赞誉。但是，要想安装arch linux是一件非常有挑战的事情。因此，使用arch是有耐心，爱挑战的人才可以享受的事情。

<!--more-->

不多说了，进入正题，首先需要制作USB-LiveCD，去archlinux官网下载[Arch linux](https://www.archlinux.org/download/)没办法FQ的小伙伴可以移步TUNA Mirror进行下载[TUNA Arch iso mirror](https://mirrors.tuna.tsinghua.edu.cn/archlinux/iso/latest/).制作安装USB-LiveCD推荐在windows下面使用[rufus](http://rufus.akeo.ie/),rufus默认配置下制作usb盘。

设置需要安装arch的电脑BIOS，引导选择为UEFI，关闭security Boot.插入USB-LiveCD，从该USB盘启动系统。

![arch boot](https://i1.piimg.com/567571/8adc9c5f8baeade5.png)

稍等片刻会进入linux shell。大arch就是这么傲娇，直接给一个shell环境，接下来自己动手解决安装问题。

首先需要却确定需要安装arch的硬盘

```sh
lsblk
```

此时会看到系统挂载的硬盘，一般硬盘编号为/sdX.我们这里假设我们需要安装的硬盘为sda。
接下来开始进行硬盘分区，很多人觉得分盘是一个非常麻烦的事情，但是呢，其实只要一步一步的来进行，其实还是很容易的。跨过了分区这一步，就成功了一半。由于我们需要使用GPT分区方案，因此要使用parted分区软件来进行分区。

首先

```sh
parted /dev/sda
```

进入parted分区软件

```sh
（parted） mklabel gpt
```

设置sda为GPT分区方案

```sh
（parted) mkpart ESP fat32 1MiB 513MiB
```

设置用于EFI引导的分区（ESP:EFI System partition），文件系统为fat32

```sh
(parted) set 1 boot on
```

设置为可启动分区

```sh
（parted) name 1 efi-partition
```

名称设置为efi-partition

```sh
(parted) mkpart primary 513MiB 1025MiB
```

创建一个主要分区，作为/boot分区

```sh
(parted) name 2 boot-partition
```

设置名称

```sh
(parted) mkpart primary 1025MiB 100%
```

这里设置一个主分区，大小为从1025MiB开始到硬盘末尾。也就是目前硬盘余下的空间都作为该分区。

```sh
（parted) name 3 lvm-partition
```

设置名称为lvm-partition

```sh
（parted) print
```

输出当前分区信息

```sh
（parted) quit
```

退出parted

接下来我们需要在分区上创建LVM和文件系统。（LVM是一种linux下对磁盘分区进行管理的一种机制）

```sh
parted /dev/sda set 3 lvm on
parted /dev/sda print
```

可以看到第三个分区flags变成了lvm。

创建物理卷

```sh
pvcreate /dev/sda2
```

创建卷管理组（volume group)

```sh
vgcreate arch-lvm /dev/sda3
```

创建逻辑卷

```sh
lvcreate -n arch-root -L 20G arch-lvm
```
```sh
lvcreate -n arch-swap -L 2G arch-lvm
```
```sh
lvcreate -n arch-home -l 100%FREE arch-lvm
```
```sh
lvs
```

这里创建了一个20G的逻辑卷，一个2G的交换空间，一个余下空间大小的逻辑卷，第一个arch-root就是安装好的linux的/目录所在，arch-home为/home目录所在。至于交换空间，建议和内存大小一样大，如果是内存足够大，同时不想让电脑休眠的，那么不设置交换空间也是可以的。

创建文件系统，我觉得尝试btrfs，此文件系统过于先进，以至于性能比较垃圾，但是，对SSD比较友好，同时，支持快照功能，现在，据说还算可以能用。可以尝试一下。

```sh
mkfs.fat -F32 /dev/sda1
mkfs.ext2 /dev/sda2
mkfs.btrfs -L root /dev/arch-lvm/arch-root
mkfs.btrfs -L home /dev/arch-lvm/arch-home
mkswap /dev/arch-lvm/arch-swap
swapon /dev/arch-lvm/arch-swap
```


这里格式化为各种适合的分区，同时，设置了交换空间。

接下来开始挂载分区

```sh
mount /dev/arch-lvm/arch-root /mnt
mkdir /mnt/{home,boot}
mount /dev/sda2 /mnt/boot
mount /dev/arch-lvm/arch-home /mnt/home
mkdir /mnt/boot/efi
mount /dev/sda1 /mnt/boot/efi
```

挂载分区到对应的目录下，请按照以上顺序执行命令。

安装基础的系统

在安装之前可以去修改一下/etc/pacman.d/mirrorlist文件，把国内的镜像源放到最前面，什么TUNA，163的源都放到前面去。然后执行

```sh
pacman -Syy
```

Ok,开始安装基础的系统

```sh
pacstrap /mnt base base-devel efibootmgr vim btrfs-progs --noconfirm
```
设置系统

```sh
genfstab -U -p /mnt > /mnt/etc/fstab
```

切换/mnt为根目录

```sh
arch-chroot /mnt /bin/bash
```

修改一下/etc/mkinitcpio.conf使得内核加载LVM和Btrfs模块

```sh
vim /etc/mkinitcpio.conf
```

找到 HOOKS=”…”，在filesystems前面加上lvm2

![lvm2](https://i1.piimg.com/567571/9f991e0ac14ed3dd.png)

接下来重新设置initrd内核镜像

```sh
mkinitcpio -p linux
```

安装grub2引导器，需要64位机器。

```sh
pacman -S grub
```

```sh
grub-install --target=x86_64-efi --efi-directory=/boot/efi \ 
--bootloader-id=grub
```

获取grub设置文件

```sh
grub-mkconfig -o /boot/grub/grub.cfg
mkdir /boot/efi/EFI/arch
```

```sh
grub-mkconfig -o  /boot/efi/EFI/arch/grub.cfg
```

设置root帐号密码，添加个人帐号

```sh
passwd root
useradd -G power,audio,wheel,storage username
passwd username
```

设置主机名称和本地化

```sh
echo "arch.localhost" > /etc/hostname
vim /etc/locale.gen
```


去掉en_US.UTF-8 UTF-8前面的#号。

现在获取本地化支持

```sh
locale-gen 
echo "LANG=en_US.UTF-8" > /etc/locale.conf
```

设置时区

```sh
rm /etc/localtime
ln -s /usr/share/zoneinfo/Asia/Shanghai  /etc/localtime
```

修改pacman配置文件打开multilib

```sh
vim /etc/pacman.conf
```

去掉

[multilib]
Include = /etc/pacman.d/mirrorlist

这两行前面的#号

安装基础软件

```sh
pacman -S bash-completion openssh zsh chromium sudo dhclient --noconfirm
```

安装X11 server

```sh
pacman -S xorg --noconfirm
```
安装音频驱动

```sh
pacman -S alsa-{utils,plugins,firmware} \
 pulseaudio pulseaudio-{equalizer,alsa,gconf} --noconfirm
```

安装触摸板以及鼠标，键盘驱动，如果没有触摸板，可以不需要synaptics

```sh
pacman -S xf86-input-{keyboard,synaptics,mouse,libinput}
```

接下来安装显卡驱动

intel:

```sh
pacman -S xf86-video-intel
```

amd:

```sh
pacman -S xf86-video-ati(老卡）
pacman -S xf86-video-amdgpu(新卡）
```

如何确定新卡老卡呢，GNC架构1代2代为老卡，3代4代为新卡。

```sh
pacman -S mesa-libgl
```

nvidia:

```sh
pacman -S nvidia nvidia-utils nvidia-settings xorg-server-devel opencl-nvidia
```

设置sudo

```sh
echo "export EDITOR=vim" >> ~/.bashrc
source ~/.bashrc
visudo
```
修改
```
# %wheel ALL=(ALL) ALL
```
为

```
%wheel ALL=(ALL) ALL
```
安装桌面环境

这里我推荐KDE，其他的桌面我不是很推荐


KDE：

```sh
pacman -S plasma kde-applications-meta
systemctl enable sddm.service
```
XFCE：
```sh
pacman -S xfce4 xfce4-goodies xfce4-mixer gstreamer0.10-good-plugins \
libxnvctrl xscreensaver
pacman -S lightdm lightdm-gtk-greeter
systemctl enable lightdm.service
```

最后：

在重启之前要做一点工作,为你的帐号加上默认home文件夹

如果安装了kde

```sh
pacman -S plasma-nm
rm /usr/share/dbus-1/system-services/org.freedesktop.NetworkManager.service
cp  /usr/lib/systemd/system/NetworkManager.service /etc/systemd/system
vi /etc/systemd/system/NetworkManager.service
去掉接下来这一行前面的#号
Alias=dbus-org.freedesktop.NetworkManager.service
```

```sh
mkdir /home/username
cp /etc/skel/.* /home/username
chown -R username:username /home/username
exit
umount -R /mnt
reboot
```


不出意外就可以进入新安装的系统了，但是可能发现不能上网，做以下一些事情，在重新启动就可以了






























