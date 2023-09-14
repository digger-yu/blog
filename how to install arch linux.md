# how to install arch linux 

 本安装环境采用vmware 
 
 镜像文件:archlinux-2023.09.01-x86_64.iso

 [推荐阅读官方安装指导](https://wiki.archlinuxcn.org/wiki/%E5%AE%89%E8%A3%85%E6%8C%87%E5%8D%97#%E5%AE%89%E8%A3%85%E5%BC%95%E5%AF%BC%E7%A8%8B%E5%BA%8F)

# 选择镜像源
```bash
reflector --country China --age 24 --sort rate --protocol https --save /etc/pacman.d/mirrorlist
pacman -Syy
cat /etc/pacman.d/mirrorlist
cp /etc/pacman.d/mirrorlist /etc/pacman.d/mirrorlist.bak
#或者手工编辑一下
Server = https://mirrors.bfsu.edu.cn/archlinux/$repo/os/$arch
Server = https://mirrors.cqu.edu.cn/archlinux/$repo/os/$arch
Server = https://mirrors.dgut.edu.cn/archlinux/$repo/os/$arch
Server = https://mirrors.neusoft.edu.cn/archlinux/$repo/os/$arch
Server = https://mirrors.nju.edu.cn/archlinux/$repo/os/$arch
Server = https://mirror.redrock.team/archlinux/$repo/os/$arch
Server = https://mirrors.sjtug.sjtu.edu.cn/archlinux/$repo/os/$arch
Server = https://mirrors.tuna.tsinghua.edu.cn/archlinux/$repo/os/$arch
Server = https://mirrors.ustc.edu.cn/archlinux/$repo/os/$arch
Server = https://mirrors.xjtu.edu.cn/archlinux/$repo/os/$arch

``````
# 硬盘准备
## 验证引导模式

```bash
要验证系统目前的引导模式，请用下列命令行出 efivars 目录：
# ls /sys/firmware/efi/efivars
如果命令结果显示了目录且没有报告错误，则系统是以 UEFI 模式引导。如果目录不存在，则系统可能是以BIOS模式（或 CSM 模式）引导。如果系统没有以您想要的模式引导启动，请您参考自己的计算机或主板说明书。
## 该项需要与后面的分区是采用UEFI还是传统bios相匹配
``````


## 分区配置


 [建议提前阅读](https://wiki.archlinuxcn.org/wiki/GRUB#GUID_%E5%88%86%E5%8C%BA%E8%A1%A8_(GPT)_%E7%89%B9%E6%AE%8A%E6%93%8D%E4%BD%9C)
 UEFI选择EF00 BIOS选择EF02

```bash
gdisk /dev/vda 
#n 新建分区
#w 保存分区
#d 删除分区
#p 打印分区
#q 退出
#+512M  ef00
#+18G 
#n 8200
lsblk 查看
mkfs.vfat /Dev/sda2
mkfs.ext4 /dev/sda3 or mkfs.vfs /dev/sda3
mkswap /dev/sda4
lsblk -f 显示分区格式
```bash
root@archiso ~ # lsblk
NAME  MAJ:MIN RM   SIZE RO TYPE MOUNTPOINTS
loop0   7:0    0 682.6M  1 loop /run/archiso/airootfs
sda     8:0    0    20G  0 disk
sr0    11:0    1 804.3M  0 rom  /run/archiso/bootmnt
Command (? for help): n
Partition number (1-128, default 1):
First sector (34-41943006, default = 2048) or {+-}size{KMGTP}:
Last sector (2048-41943006, default = 41940991) or {+-}size{KMGTP}: +1M
Current type is 8300 (Linux filesystem)
Hex code or GUID (L to show codes, Enter = 8300): EF02
Changed type of partition to 'BIOS boot partition'

Command (? for help): N
Partition number (2-128, default 2):
First sector (34-41943006, default = 4096) or {+-}size{KMGTP}:
Last sector (4096-41943006, default = 41940991) or {+-}size{KMGTP}: +512M
Current type is 8300 (Linux filesystem)
Hex code or GUID (L to show codes, Enter = 8300): EF02
Changed type of partition to 'BIOS boot partition'

Command (? for help): N
Partition number (3-128, default 3):
First sector (34-41943006, default = 1052672) or {+-}size{KMGTP}:
Last sector (1052672-41943006, default = 41940991) or {+-}size{KMGTP}: +18G
Current type is 8300 (Linux filesystem)
Hex code or GUID (L to show codes, Enter = 8300):
Changed type of partition to 'Linux filesystem'

Command (? for help): N
Partition number (4-128, default 4):
First sector (34-41943006, default = 38801408) or {+-}size{KMGTP}:
Last sector (38801408-41943006, default = 41940991) or {+-}size{KMGTP}:
Current type is 8300 (Linux filesystem)
Hex code or GUID (L to show codes, Enter = 8300): 8200
Changed type of partition to 'Linux swap'

Command (? for help): P
Disk /dev/sda: 41943040 sectors, 20.0 GiB
Model: VMware Virtual S
Sector size (logical/physical): 512/512 bytes
Disk identifier (GUID): 1A72A899-DDCF-4052-94E5-412EE74516E7
Partition table holds up to 128 entries
Main partition table begins at sector 2 and ends at sector 33
First usable sector is 34, last usable sector is 41943006
Partitions will be aligned on 2048-sector boundaries
Total free space is 4029 sectors (2.0 MiB)

Number  Start (sector)    End (sector)  Size       Code  Name
   1            2048            4095   1024.0 KiB  EF02  BIOS boot partition
   2            4096         1052671   512.0 MiB   EF02  BIOS boot partition
   3         1052672        38801407   18.0 GiB    8300  Linux filesystem
   4        38801408        41940991   1.5 GiB     8200  Linux swap

Command (? for help): W

Final checks complete. About to write GPT data. THIS WILL OVERWRITE EXISTING
PARTITIONS!!

Do you want to proceed? (Y/N): Y
OK; writing new GUID partition table (GPT) to /dev/sda.
The operation has completed successfully.


#################
```


## 格式化分区
```bash
root@archiso ~ #  lsblk
NAME   MAJ:MIN RM   SIZE RO TYPE MOUNTPOINTS
loop0    7:0    0 682.6M  1 loop /run/archiso/airootfs
sda      8:0    0    20G  0 disk
├─sda1   8:1    0     1M  0 part
├─sda2   8:2    0   512M  0 part
├─sda3   8:3    0    18G  0 part
└─sda4   8:4    0   1.5G  0 part
sr0     11:0    1 804.3M  0 rom  /run/archiso/bootmnt
注意: sda1 不需要格式化

root@archiso ~ # mkfs.fat /dev/sda2
mkfs.fat 4.2 (2021-01-31)
root@archiso ~ # mkfs.ext4 /dev/sda3
mke2fs 1.47.0 (5-Feb-2023)
Creating filesystem with 4718592 4k blocks and 1179648 inodes
Filesystem UUID: 3f7c3fa7-637a-482d-a630-659dd8b730e6
Superblock backups stored on blocks:
        32768, 98304, 163840, 229376, 294912, 819200, 884736, 1605632, 2654208,
        4096000

Allocating group tables: done
Writing inode tables: done
Creating journal (32768 blocks): done
Writing superblocks and filesystem accounting information: done

root@archiso ~ # mkswap /dev/sda4
Setting up swapspace version 1, size = 1.5 GiB (1607462912 bytes)
no label, UUID=2581098f-a479-465b-a15d-dad5508d360a

root@archiso ~ #  lsblk -f
NAME   FSTYPE   FSVER            LABEL       UUID                                 FSAVAIL FSUSE% MOUNTPOINTS
loop0  squashfs 4.0                                                                     0   100% /run/archiso/airootfs
sda
├─sda1
├─sda2 vfat     FAT32                        18F0-3354
├─sda3 ext4     1.0                          3f7c3fa7-637a-482d-a630-659dd8b730e6
└─sda4 swap     1                            2581098f-a479-465b-a15d-dad5508d360a
sr0    iso9660  Joliet Extension ARCH_202309 2023-09-01-10-45-56-00                     0   100% /run/archiso/bootmnt
root@archiso ~ #



``````

## 挂载分区

```bash
root@archiso ~ # mount /dev/sda3 /mnt
root@archiso ~ # mkdir -p /mnt/boot
root@archiso ~ # mount /dev/sda2 /mnt/boot
root@archiso ~ # swapon /dev/sda4
root@archiso ~ # lsblk
NAME   MAJ:MIN RM   SIZE RO TYPE MOUNTPOINTS
loop0    7:0    0 682.6M  1 loop /run/archiso/airootfs
sda      8:0    0    20G  0 disk
├─sda1   8:1    0     1M  0 part
├─sda2   8:2    0   512M  0 part /mnt/boot
├─sda3   8:3    0    18G  0 part /mnt
└─sda4   8:4    0   1.5G  0 part [SWAP]
sr0     11:0    1 804.3M  0 rom  /run/archiso/bootmnt
root@archiso ~ #



``````

# 安装系统
```bash
root@archiso ~ # pacstrap /mnt linux linux-firmware linux-headers base base-devel vim bash-completion net-tools
==> Creating install root at /mnt
==> Installing packages to /mnt
:: Synchronizing package databases...
 core is up to date
 extra is up to date
resolving dependencies...
:: There are 3 providers available for initramfs:
:: Repository core
   1) mkinitcpio
:: Repository extra
   2) booster  3) dracut

Enter a number (default=1):
looking for conflicting packages...

Packages (153)
......
Total Download Size:    530.23 MiB
Total Installed Size:  1397.47 MiB

:: Proceed with installation? [Y/n]
:: Retrieving packages...
......
pacstrap /mnt linux linux-firmware linux-headers base base-devel vim    22.22s user 38.88s system 11% cpu 9:07.99 total

```

## 生成文件系统表文件
```bash
genfstab -U /mnt 
genfstab -U /mnt >> /mnt/etc/fstab
cat /mnt/etc/fstab

root@archiso ~ # genfstab -U /mnt
# /dev/sda3
UUID=3f7c3fa7-637a-482d-a630-659dd8b730e6       /               ext4            rw,relatime     0 1

# /dev/sda2
UUID=18F0-3354          /boot           vfat            rw,relatime,fmask=0022,dmask=0022,codepage=437,iocharset=ascii,shortname=mixed,utf8,errors=remount-ro   0 2

# /dev/sda4
UUID=2581098f-a479-465b-a15d-dad5508d360a       none            swap            defaults        0 0

root@archiso ~ # genfstab -U /mnt >> /mnt/etc/fstab
root@archiso ~ # 
 
``````

## 进入新系统
```bash
root@archiso ~ # arch-chroot /mnt

确认源
[root@archiso /]# cat /etc/pacman.d/mirrorlist
################################################################################
################# Arch Linux mirrorlist generated by Reflector #################
################################################################################

# With:       reflector --country China --sort rate --save /etc/pacman.d/mirrorlist
# When:       2023-09-12 00:36:23 UTC
# From:       https://archlinux.org/mirrors/status/json/
# Retrieved:  2023-09-12 00:35:53 UTC
# Last Check: 2023-09-11 23:41:36 UTC

Server = http://mirrors.163.com/archlinux/$repo/os/$arch
Server = https://mirrors.aliyun.com/archlinux/$repo/os/$arch
Server = https://mirrors.bfsu.edu.cn/archlinux/$repo/os/$arch
Server = https://mirrors.cqu.edu.cn/archlinux/$repo/os/$arch
Server = https://mirrors.dgut.edu.cn/archlinux/$repo/os/$arch
Server = https://mirrors.neusoft.edu.cn/archlinux/$repo/os/$arch
Server = https://mirrors.nju.edu.cn/archlinux/$repo/os/$arch
Server = https://mirror.redrock.team/archlinux/$repo/os/$arch
Server = https://mirrors.sjtug.sjtu.edu.cn/archlinux/$repo/os/$arch
Server = https://mirrors.tuna.tsinghua.edu.cn/archlinux/$repo/os/$arch
Server = https://mirrors.ustc.edu.cn/archlinux/$repo/os/$arch
Server = https://mirrors.xjtu.edu.cn/archlinux/$repo/os/$arch
[root@archiso /]# pacman -Syy
:: Synchronizing package databases...
 core                                     129.3 KiB   129 KiB/s 00:01 [#######################################] 100%
 extra                                      8.3 MiB   326 KiB/s 00:26 [#######################################] 100%
[root@archiso /]#

``````
## pacman 多线程下载
```bash
vim /etc/pacman.conf  line 37
 ParallelDownloads = 5 #5是并行下载数量，你也可以设置成其他任何数字
``````

## 新系统安装必要软件包

```bash
[root@archiso /]# pacman -S archlinux-keyring grub efibootmgr efivar networkmanager dhcp intel-ucode os-prober openssh

### 如果是amd的cpu就装amd-ucode
resolving dependencies...
looking for conflicting packages...

Packages (24) 
......
Total Download Size:    25.48 MiB
Total Installed Size:  109.46 MiB
:: Proceed with installation? [Y/n] y
:: Retrieving packages...
......
[root@archiso /]#


``````

## grub 安装
```bash
## grub 写入系统
[root@archiso pacman.d]# grub-install  /dev/sda
Installing for i386-pc platform.
Installation finished. No error reported.
[root@archiso pacman.d]#
报错就不用继续了,必然是有问题的
#grub-install --target=x86_64-efi /dev/sda
#grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=GRUB
vim /etc/default/grub
可以修改一下等待时间
GRUB_DISABLE_OS_PROBER=false 取消注释该行 line 63

## 生成grub文件
[root@archiso /]# grub-mkconfig -o /boot/grub/grub.cfg
Generating grub configuration file ...
Found linux image: /boot/vmlinuz-linux
Found initrd image: /boot/intel-ucode.img /boot/initramfs-linux.img
Found fallback initrd image(s) in /boot:  intel-ucode.img initramfs-linux-fallback.img
Warning: os-prober will be executed to detect other bootable partitions.
Its output will be used to detect bootable binaries on them and create new boot entries.
Adding boot menu entry for UEFI Firmware Settings ...
done




[推荐阅读](https://wiki.archlinux.org/title/GRUB#Installation_2)

```
## 设置root用户密码
```bash
[root@archiso /]# passwd
New password:
Retype new password:
passwd: password updated successfully

```
## 时区 语言
```bash
未安装字体前,语言可以不设置
vim /etc/locale.gen -> zh_CN.UTF-8 UTF-8
locale-gen
echo "LANG=zh_CN.UTF-8" >> /etc/locale.conf


ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
hwclock --systohc
echo "arch" >> /etc/hostname

终端字体
vim /etc/vconsole.conf
FONT=ter-132b
``````
## 连接网络
```bash
systemctl enable NetworkManager
echo "PermitRootLogin yes" >> /etc/ssh/sshd_config
systemctl enable sshd.service
```
## 卸载安装介质

```bash
exit到iso
#swapoff /mnt/swapfile
umount /mnt/boot
umount /mnt
lsblk
```

## reboot
```bash
reboot
```
## alias
```bash
vim /etc/skel/.bashrc

export EDITOR=vim
alias ll='ls -al'
alias vi='vim'
alias grep='grep --color=auto'
alias egrep='egrep --color=auto'
alias fgrep='fgrep --color=auto'

cp -a . ~

```

## add user
```bash
useradd --create-home test
passwd test

usermod -aG wheel,users,storage,power,lp,adm,optical test
id test
visudo
%wheel ALL=(ALL) ALL 取消注释 允许用户sudo
``````

## 时间同步
```bash
pacman -S ntp
timedatectl set-timezone Asia/Shanghai
ntpdate ntp.aliyun.com
timedatectl set-ntp true
timedatectl status
``````





# Easy Install

try  archinstall script 
and read [Installation_guide](https://wiki.archlinux.org/title/Installation_guide)
[pacman](https://wiki.archlinuxcn.org/wiki/Pacman)

iso 文件中的archinstall 最好更新一下
pacman -S archinstall

可以用github上面的archinstall脚本
https://github.com/archlinux/archinstall

注意:截止2023.9.13 github上的last version 2.6.0 仍存在部分问题可能导致部分机型安装后启动失败

```bash
# 脚本安装时建议增加的other packages
linux-headers  vim bash-completion net-tools grub efibootmgr efivar networkmanager dhcp intel-ucode openssh
```