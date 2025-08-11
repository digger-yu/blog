# 1. 传统 BIOS 引导模式安装步骤​​
[官方安装手册：https://wiki.archlinux.org/title/Installation_guide]
## ​​1.1 准备启动介质​​
下载 Arch Linux ISO 镜像并制作启动盘（推荐使用 dd命令或 Rufus 工具）。
确保虚拟机或物理机 BIOS 设置为 ​​Legacy BIOS 模式​​，硬盘分区表为 ​​MBR​​   
查看系统是bios还是UEFI
vmware 虚拟机在选项，高级，设置
VirtualBox 在系统页设置
```bash
[ -d /sys/firmware/efi ] && echo UEFI || echo BIOS
```
```dotnetcli
root@archiso ~ # [ -d /sys/firmware/efi ] && echo UEFI || echo BIOS
BIOS
root@archiso ~ # 

```


## 1.2 分区表
```bash
fdisk /dev/sda  # 进入分区工具
# 操作步骤：
# m → 查看帮助
# o → 创建 MBR 分区表
# n → 新建分区（/boot 分区 200MB，类型 83 Linux）
# n → 新建根分区（剩余空间，类型 83 Linux）
# a → 设置 /boot 分区为可启动
# w → 保存并退出
```
验证分区：fdisk -l /dev/sda
```dotnetcli
root@archiso ~ # fdisk -l
Disk /dev/sda: 20 GiB, 21474836480 bytes, 41943040 sectors
Disk model: VMware Virtual S
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes


Disk /dev/loop0: 959.75 MiB, 1006374912 bytes, 1965576 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
127 root@archiso ~ # fdisk /dev/sda

Welcome to fdisk (util-linux 2.41.1).
Changes will remain in memory only, until you decide to write them.
Be careful before using the write command.


Command (m for help): m

Help:

  DOS (MBR)
   a   toggle a bootable flag
   b   edit nested BSD disklabel
   c   toggle the dos compatibility flag

  Generic
   d   delete a partition
   F   list free unpartitioned space
   l   list known partition types
   n   add a new partition
   p   print the partition table
   t   change a partition type
   v   verify the partition table
   i   print information about a partition
   e   resize a partition
   T   discard (trim) sectors

  Misc
   m   print this menu
   u   change display/entry units
   x   extra functionality (experts only)

  Script
   I   load disk layout from sfdisk script file
   O   dump disk layout to sfdisk script file

  Save & Exit
   w   write table to disk and exit
   q   quit without saving changes

  Create a new label
   g   create a new empty GPT partition table
   G   create a new empty SGI (IRIX) partition table
   o   create a new empty MBR (DOS) partition table
   s   create a new empty Sun partition table


Command (m for help): o
Created a new DOS (MBR) disklabel with disk identifier 0x1b0ee54a.

Command (m for help): n
Partition type
   p   primary (0 primary, 0 extended, 4 free)
   e   extended (container for logical partitions)
Select (default p): 

Using default response p.
Partition number (1-4, default 1): 
First sector (2048-41943039, default 2048): 
Last sector, +/-sectors or +/-size{K,M,G,T,P} (2048-41943039, default 41943039): +512M

Created a new partition 1 of type 'Linux' and of size 512 MiB.

Command (m for help): n
Partition type
   p   primary (1 primary, 0 extended, 3 free)
   e   extended (container for logical partitions)
Select (default p): 

Using default response p.
Partition number (2-4, default 2): 
First sector (1050624-41943039, default 1050624): 
Last sector, +/-sectors or +/-size{K,M,G,T,P} (1050624-41943039, default 41943039): 

Created a new partition 2 of type 'Linux' and of size 19.5 GiB.

Command (m for help): p
Disk /dev/sda: 20 GiB, 21474836480 bytes, 41943040 sectors
Disk model: VMware Virtual S
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0x1b0ee54a

Device     Boot   Start      End  Sectors  Size Id Type
/dev/sda1          2048  1050623  1048576  512M 83 Linux
/dev/sda2       1050624 41943039 40892416 19.5G 83 Linux

Command (m for help): w
The partition table has been altered.
Calling ioctl() to re-read partition table.
Syncing disks.

root@archiso ~ #
```

## 1.3 格式化分区
```bash
mkfs.ext4 /dev/sda1  # 格式化 /boot 分区
mkfs.ext4 /dev/sda2  # 格式化根分区
mount /dev/sda2 /mnt  # 挂载根分区
mount --mkdir /dev/sda1 /mnt/boot  # 挂载 /boot 分区
```
## 1.4 源配置
```bash
systemctl stop reflector
curl -L 'https://archlinux.org/mirrorlist/?country=CN&protocol=https' -o /etc/pacman.d/mirrorlist
vim /etc/pacman.d/mirrorlist

# 修改 /etc/pacman.conf
vim /etc/pacman.d/mirrorlist




[options]
# 启用并行下载（默认单线程）
ParallelDownloads = 10  # 默认同时下载 5 个包
# 启用 Color 输出
Color
# 启用 verbose 模式
#VerbosePkgLists
# 启用包压缩
#CompressXZ
# 启用包签名
#SigLevel = Required DatabaseOptional
#pacman -S aria2
#vi /etc/pacman.conf
# 启用 aria2 下载
#XferCommand = /usr/bin/aria2c -s 16 -m 2 -d / -o %o %u
#XferCommand = /usr/bin/curl -L -C - -f -o %o %u
#XferCommand = /usr/bin/wget --passive-ftp -c -O %o %u

pacman -Sy pacman-mirrorlist
```    


## 1.5 安装基本系统   

```dotnetcli
pacstrap /mnt base base-devel linux linux-firmware grub dhcpcd networkmanager vim openssh iwd
genfstab -U /mnt >> /mnt/etc/fstab  # 生成 fstab 文件
```
## 1.6 配置时区与语言 主机名 网络
```dotnetcli
# 进入 chroot 环境
arch-chroot /mnt 
# 设置时区
ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime 
# 同步硬件时间 
hwclock --systohc  
# 生成 locale
cp /etc/locale.gen /etc/locale.gen.bak
echo "en_US.UTF-8 UTF-8" >> /etc/locale.gen 
locale-gen

# 设置 locale 
echo "LANG=en_US.UTF-8" > /etc/locale.conf 
##  配置主机名网络
echo "myarch" > /etc/hostname
systemctl enable dhcpcd
echo "127.0.0.1 localhost" >> /etc/hosts
echo "::1       localhost" >> /etc/hosts
echo "127.0.1.1 myarch.localdomain myarch" >> /etc/hosts
# 设置密码
passwd        
```
## 1.7 安装grub

```bash
pacman -S grub intel-ucode
#根据cpu选择安装intel-ucode或amd-ucode,双系统选择os-prober

grub-install --target=i386-pc /dev/sda  # 安装 GRUB
grub-mkconfig -o /boot/grub/grub.cfg  # 生成 GRUB 配置文件
systemctl enable NetworkManager
```
## 1.8 配置引导
```bash
#echo "GRUB_TIMEOUT=5" >> /etc/default/grub
#echo "GRUB_DISTRIBUTOR=\"Arch\"" >> /etc/default/grub
#echo "GRUB_DEFAULT=0" >> /etc/default/grub
#echo "GRUB_HIDDEN_TIMEOUT=0" >> /etc/default/grub
#echo "GRUB_HIDDEN_TIMEOUT_QUIET=true" >> /etc/default/grub    
```

## 1.9 重启
```dotnetcli
exit
umount -R /mnt
reboot
```
```dotnetcli
systemctl enable NetworkManager
systemctl start NetworkManager
```


# 2. UEFI模式安装步骤
```dotnetcli
[root@myarch ~]# [ -d /sys/firmware/efi ] && echo UEFI || echo BIOS
UEFI
[root@myarch ~]#

```


## 2.1 分区表
```bash
fdisk /dev/sda  # 进入分区工具
# 操作步骤：
# m → 查看帮助
# g → 创建 GPT 分区表
# n → 新建分区（/boot 分区 200MB，类型 83 Linux）
# n → 新建根分区（剩余空间，类型 83 Linux）
# a → 设置 /boot 分区为可启动
# w → 保存并退出
```
```dotnetcli
Disk /dev/sda: 20 GiB, 21474836480 bytes, 41943040 sectors
Disk model: VMware Virtual S
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: gpt
Disk identifier: EE006AB3-0134-48A8-B515-FFD77534D0A9

Device       Start      End  Sectors  Size Type
/dev/sda1     2048  1050623  1048576  512M Linux filesystem
/dev/sda2  3147776 41940991 38793216 18.5G Linux filesystem
/dev/sda3  1050624  3147775  2097152    1G Linux filesystem

Partition table entries are not in disk order.

Command (m for help): w

```

验证分区：fdisk -l /dev/sda

## 2.2 格式化分区
```bash
mkfs.fat -F32 /dev/sda1  # 格式化 /boot 分区
mkfs.ext4 /dev/sda2  # 格式化根分区
mkswap /dev/sda3（交换空间分区）
swapon /dev/sda3

mount /dev/sda2 /mnt  # 挂载根分区
mount --mkdir /dev/sda1 /mnt/boot  # 挂载 /boot 分区
```
## 2.3 源配置
```bash
systemctl stop reflector
curl -L 'https://archlinux.org/mirrorlist/?country=CN&protocol=https' -o /etc/pacman.d/mirrorlist
vim /etc/pacman.d/mirrorlist
pacman -Sy pacman-mirrorlist
# 修改 /etc/pacman.conf
[options]
# 启用并行下载（默认单线程）
ParallelDownloads = 15  # 默认同时下载 5 个包,修改后感觉区别不大
# 启用 Color 输出
Color
# 启用 verbose 模式
#VerbosePkgLists
# 启用包压缩
#CompressXZ
# 启用包签名
#SigLevel = Required DatabaseOptional

```    


## 2.4 安装基本系统   

```dotnetcli
pacstrap /mnt base base-devel linux linux-firmware grub dhcpcd networkmanager vim openssh iwd
genfstab -U /mnt >> /mnt/etc/fstab  # 生成 fstab 文件
```
## 2.5 配置时区与语言 主机名 网络
```dotnetcli
# 进入 chroot 环境
arch-chroot /mnt 
# 设置时区
ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime 
# 同步硬件时间 
hwclock --systohc  
# 生成 locale
cp /etc/locale.gen /etc/locale.gen.bak
echo "en_US.UTF-8 UTF-8" >> /etc/locale.gen 
locale-gen
# 设置 locale 
echo "LANG=en_US.UTF-8" > /etc/locale.conf 
##  配置主机名网络
echo "myarch" > /etc/hostname
systemctl enable dhcpcd
echo "127.0.0.1 localhost" >> /etc/hosts
echo "::1       localhost" >> /etc/hosts
echo "127.0.1.1 myarch.localdomain myarch" >> /etc/hosts
# 设置密码
passwd        
```
## 2.6 安装grub

```bash
pacman -S grub efibootmgr intel-ucode os-prober
#根据cpu选择安装intel-ucode或amd-ucode,双系统选择os-prober

grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=GRUB  # 安装 GRUB
grub-mkconfig -o /boot/grub/grub.cfg  # 生成 GRUB 配置文件
```

## 2.7 重启
```dotnetcli
exit
umount -R /mnt
reboot
```
```dotnetcli
sudo systemctl enable NetworkManager
sudo systemctl start NetworkManager
```