
# ISO
安装源地址 http://mirrors.aliyun.com/centos/7/os/x86_64/   
系统镜像   https://mirrors.aliyun.com/centos/7.9.2009/isos/x86_64/CentOS-7-x86_64-DVD-2207-02.iso

ctrl+alt+f2 进入终端, 卸载首次安装后的初始化流程
```
yum remove gnome-initial-setup.x86_64
reboot
```

# 更改yum源  
```
curl -o /etc/yum.repos.d/CentOS-Base.repo https://mirrors.aliyun.com/repo/Centos-7.repo
```

#安装软件包
```bash
yum -y install dhcp xinetd tftp tftp-server httpd 
yum -y install system-config-kickstart
yum -y install syslinux
```
# 准备
```bash
systemctl stop firewalld
systemctl disable firewalld
setenforce 0
vi /etc/selinux/config
disabled
vim /etc/resolv.conf
vim /etc/sysconfig/network-scripts/ifcfg-ens160
```

# 网卡配置固定ip <br>
vmware 虚拟机里面如果网卡选择vmnet8,需要关闭dhcp

```bash
[root@localhost ~]# cd /etc/sysconfig/network-scripts/
[root@localhost network-scripts]# cat ifcfg-ens160 
TYPE=Ethernet
PROXY_METHOD=none
BROWSER_ONLY=no
BOOTPROTO=none
DEFROUTE=yes
IPV4_FAILURE_FATAL=no
IPV6INIT=yes
IPV6_AUTOCONF=yes
IPV6_DEFROUTE=yes
IPV6_FAILURE_FATAL=no
IPV6_ADDR_GEN_MODE=stable-privacy
NAME=ens160
UUID=a7045709-3285-4007-958e-bb6df7486da1
DEVICE=ens160
ONBOOT=yes
IPADDR=192.168.7.100
PREFIX=24
GATEWAY=192.168.7.2
DNS1=192.168.7.2
[root@localhost network-scripts]# 
```
# dhcp 配置
```bash
vi /etc/dhcp/dhcpd.conf 
subnet 192.168.7.0 netmask 255.255.255.0 {
    range 192.168.7.50 192.168.7.100;
    option routers 192.168.7.2;
    option broadcast-address 192.168.7.255;
    option domain-name-servers 223.5.5.5, 8.8.4.4;
    default-lease-time 600;
    max-lease-time 7200;
    filename "pxelinux.0";
    next-server 192.168.7.100;
}
```
# pxe repo 配置
```bash
vi /etc/yum.repos.d/pxe.repo
[development]
name=pxe
baseurl=http://192.168.7.100/pub
enabled=1
gpgcheck=0
```

```bash
Systemctl restart httpd
mkdir /var/www/html/pub
挂载镜像
mount /dev/cdrom /var/www/html/pub
 service httpd restart
```

# tftp 配置
```bash
vi /etc/xinetd.d/tftp
# default: off
# description: The tftp server serves files using the trivial file transfer \
#        protocol.  The tftp protocol is often used to boot diskless \
#        workstations, download configuration files to network-aware printers, \
#        and to start the installation process for some operating systems.
service tftp
{
        socket_type                = dgram
        protocol                = udp
        wait                        = yes
        user                        = root
        server                        = /usr/sbin/in.tftpd
        server_args                = -s /var/lib/tftpboot
        disable                        = no
        per_source                = 11
        cps                        = 100 2
        flags                        = IPv4
}
```
```bash
service xinetd restart
systemctl restart xinetd.service
systemctl restart tftp
systemctl restart tftp.socket
```

拷贝文件
```bash
cp /usr/share/syslinux/pxelinux.0 /var/lib/tftpboot/

mkdir /var/lib/tftpboot/pxelinux.cfg
cp /var/www/html/pub/isolinux/isolinux.cfg /var/lib/tftpboot/pxelinux.cfg/default 
```
# pxelinux.cfg/default
主要修改61-71行 label linux 处
vi /var/lib/tftpboot/pxelinux.cfg/default
```bash
default vesamenu.c32
timeout 600

display boot.msg

# Clear the screen when exiting the menu, instead of leaving the menu displayed.
# For vesamenu, this means the graphical background is still displayed without
# the menu itself for as long as the screen remains in graphics mode.
menu clear
menu background splash.png
menu title CentOS 7
menu vshift 8
menu rows 18
menu margin 8
#menu hidden
menu helpmsgrow 15
menu tabmsgrow 13

# Border Area
menu color border * #00000000 #00000000 none

# Selected item
menu color sel 0 #ffffffff #00000000 none

# Title bar
menu color title 0 #ff7ba3d0 #00000000 none

# Press [Tab] message
menu color tabmsg 0 #ff3a6496 #00000000 none

# Unselected menu item
menu color unsel 0 #84b8ffff #00000000 none

# Selected hotkey
menu color hotsel 0 #84b8ffff #00000000 none

# Unselected hotkey
menu color hotkey 0 #ffffffff #00000000 none

# Help text
menu color help 0 #ffffffff #00000000 none

# A scrollbar of some type? Not sure.
menu color scrollbar 0 #ffffffff #ff355594 none

# Timeout msg
menu color timeout 0 #ffffffff #00000000 none
menu color timeout_msg 0 #ffffffff #00000000 none

# Command prompt text
menu color cmdmark 0 #84b8ffff #00000000 none
menu color cmdline 0 #ffffffff #00000000 none

# Do not display the actual menu unless the user presses a key. All that is displayed is a timeout message.

menu tabmsg Press Tab for full configuration options on menu items.

menu separator # insert an empty line
menu separator # insert an empty line

label linux
  menu label ^Install CentOS 7
  kernel vmlinuz
  append initrd=initrd.img ks=http://192.168.7.100/ks/ks123.cfg
  menu default

menu separator # insert an empty line

# utilities submenu
menu begin ^Troubleshooting
  menu title Troubleshooting

label vesa
  menu indent count 5
  menu label Install CentOS 7 in ^basic graphics mode
  text help
        Try this option out if you're having trouble installing
        CentOS 7.
  endtext
  kernel vmlinuz
  append initrd=initrd.img inst.stage2=hd:LABEL=CentOS\x207\x20x86_64 xdriver=vesa nomodeset quiet

label rescue
  menu indent count 5
  menu label ^Rescue a CentOS system
  text help
        If the system will not boot, this lets you access files
        and edit config files to try to get it booting again.
  endtext
  kernel vmlinuz
  append initrd=initrd.img inst.stage2=hd:LABEL=CentOS\x207\x20x86_64 rescue quiet

label memtest
  menu label Run a ^memory test
  text help
        If your system is having issues, a problem with your
        system's memory may be the cause. Use this utility to
        see if the memory is working correctly.
  endtext
  kernel memtest

menu separator # insert an empty line

label local
  menu label Boot from ^local drive
  localboot 0xffff

menu separator # insert an empty line
menu separator # insert an empty line

label returntomain
  menu label Return to ^main menu
  menu exit

menu end
```

# 创建ks目录
 mkdir /var/www/html/ks   
 vmware虚拟机图形界面执行   
system-config-kickstart   
![alt text](img\pxe\image-1.png)  
![alt text](img\pxe\image-2.png)   
![alt text](img\pxe\image-3.png)   
![alt text](img\pxe\image-4.png)   
![alt text](img\pxe\image-5.png)   
![alt text](img\pxe\image-6.png)   
拷贝ks配置到ks目录
```bash
cp ks123.cfg /var/www/html/ks/
cp /var/www/html/pub/isolinux/* /var/lib/tftpboot/
```

# ks123.cfg
```bash

#platform=x86, AMD64, 或 Intel EM64T
#version=DEVEL
# Install OS instead of upgrade
install
# Keyboard layouts
keyboard 'us'
# Root password
rootpw --plaintext 123
# System language
lang zh_CN
# System authorization information
auth  --useshadow  --passalgo=sha512
# Use text mode install
text
# SELinux configuration
selinux --disabled
# Do not configure the X Window System
skipx


# Firewall configuration
firewall --disabled
# Network information
network  --bootproto=dhcp --device=eth0
# Reboot after installation
reboot
# System timezone
timezone Asia/Shanghai
# Use network installation
url --url="http://192.168.7.100/pub"
# System bootloader configuration
bootloader --location=mbr
# Clear the Master Boot Record
zerombr
# Partition clearing information
clearpart --all --initlabel
# Disk partitioning information
part / --fstype="xfs" --size=10240
part /boot --fstype="xfs" --size=200

%post
#! /bin/bash
cd root
mkdir 123
curl -o /etc/yum.repos.d/Centos-Base.repo https://mirrors.aliyun.com/repo/Centos-7.repo
%end

%packages
@base
@system-management

%end
```

done.