# 1. 安装准备
安装后修改软件源为aliyun
```dotnetcli
sudo apt-get install ssh vim net-tools 
sudo systemctl enable ssh
```
## 1.2 修改网卡名
```dotnetcli
修改ubuntu 网卡名为eth0 
1.修改grub文件
vim /etc/default/grub
查找
GRUB_CMDLINE_LINUX=""
修改为
GRUB_CMDLINE_LINUX="net.ifnames=0 biosdevname=0"
2.重新生成grub引导配置文件
grub-mkconfig -o /boot/grub/grub.cfg
```
### 1.2.2 固定网卡号
```dotnetcli
## ethtools -i eth* 查看并记录对应的bus-info

vi /etc/udev/rules.d/10-rename-network.rules
SUBSYSTEM=="net", ACTION=="add", ATTR{address}=="f4:6d:2f:de:29:09", NAME="eth0"
SUBSYSTEM=="net", ACTION=="add", ATTR{address}=="a0:36:9f:21:a8:88", NAME="eth1"
SUBSYSTEM=="net", ACTION=="add", ATTR{address}=="50:9a:4c:08:15:e9", NAME="eth2"
SUBSYSTEM=="net", ACTION=="add", ATTR{address}=="00:1b:21:bd:de:be", NAME="eth3"
SUBSYSTEM=="net", ACTION=="add", ATTR{address}=="00:1b:21:bd:de:bf", NAME="eth4"
```
## 1.3 虚拟机网卡设置
![Alt text](/img/image.png)

给虚拟机添加两块网卡，分别桥接在物理机两块不同的网卡上面 我这里是VMnet0 和VMnet2
VMnet0 主要用于虚拟机访问外网
VMnet2 注意用于连接路由器wan口，用于dhcp，pppoe vpn 等的server 
添加完成后，在Vmware上打开“虚拟机”——>“设置”页面，查看两个网口网络适配器处网络连接，均应为自定义状态（比如选择为桥接模式，就无法给下挂终端分配地址，多次连续重启虚拟机后可能出现该配置项配置变化的情况）

## 1.4 ip地址规划
```dotnetcli
内网网卡地址设置为192.168.1.1
pppoe拨号地址设置为192.168.1.1
vpn服务器地址设置为20.1.1.1
通过图形界面配置的ip，具体配置文件路径如下
root@ubuntu:~# cat /etc/NetworkManager/system-connections/有线连接\ 2
[connection]
id=有线连接 2
uuid=f323aa35-4201-3bcd-b0e3-4971ef943aee
type=ethernet
autoconnect-priority=-999
permissions=
timestamp=1671707687

[ethernet]
mac-address=00:0C:29:9D:C4:53
mac-address-blacklist=

[ipv4]
address1=192.168.1.1/24
address2=20.1.1.1/24
dns-search=
method=manual

[ipv6]
addr-gen-mode=stable-privacy
address1=2001:1::1/64
dns-search=
method=manual
查看网卡uuid
root@ubuntu:~# nmcli con
NAME        UUID                                  TYPE      DEVICE
有线连接 1  9e953b8b-99d3-3cf4-84de-603714ceff91  ethernet  eth0
有线连接 2  f323aa35-4201-3bcd-b0e3-4971ef943aee  ethernet  eth1
root@ubuntu:~#


sudo service network-manager restart
```
ipv6 

https://www.iana.org/assignments/ipv6-address-space/ipv6-address-space.xhtml

IPv6网络地址后面跟着的 /48，/32指的是ipv6地址的前缀长度（即前48或32位长度的地址相同）。
当ipv6地址是/48的时候，128-48-64=16位，即可用的ip数是FFFF(十六进制表示法，即65535)，可理解为可分配65535个家庭或公司。
当ipv6地址是/32的时候，128-32-64=32位，即可用ip数是FFFFFFFF(约等于42.95亿，即可分配42.95亿个家庭或公司。
其实，只需一个/64前缀的地址就已经完全足够整个中国家庭使用，2的64次方总数为1844亿亿个地址，就算去掉一个亿字，都还有1844亿

## 1.5 修改内核参数
```dotnetcli
vi如下文件时，在文件对应位置取消注释符号，
sudo vi /etc/sysctl.conf
net.ipv4.ip_forward = 1
net.ipv6.conf.all.forwarding = 1
net.ipv6.conf.all.proxy_ndp=1

sudo sysctl -p /etc/sysctl.conf
sysctl -a |grep ipv6 |grep forward
https://www.eukhost.com/kb/how-to-enable-ip-forwarding-on-linux-ipv4-ipv6/
##net.ipv6.conf.all.accept_ra = 1
```

```dotnetcli
https://www.kernel.org/doc/Documentation/networking/ip-sysctl.txt

accept_ra - INTEGER
        Accept Router Advertisements; autoconfigure using them.

        It also determines whether or not to transmit Router
        Solicitations. If and only if the functional setting is to
        accept Router Advertisements, Router Solicitations will be
        transmitted.

        Possible values are:
                0 Do not accept Router Advertisements.
                1 Accept Router Advertisements if forwarding is disabled.
                2 Overrule forwarding behaviour. Accept Router Advertisements
                  even if forwarding is enabled.

        Functional default: enabled if local forwarding is disabled.
                            disabled if local forwarding is enabled.

```
```dotnetcli
##net.ipv6.conf.all.proxy_ndp = 1 ？？
net.ipv6.conf.all.forwarding=1 
net.ipv6.conf.eth1.accept_ra=2 

net.ipv6.conf.eth1.accept_ra_defrtr=1 
net.ipv6.conf.eth1.router_solicitations=1

cat /proc/sys/net/ipv6/conf/eth1/accept_ra
https://tldp.org/HOWTO/Linux+IPv6-HOWTO/ch11s02.html
https://linux.die.net/man/8/ip6tables
https://sysctl-explorer.net/net/ipv6/
```
## 1.6 Enable IPV4&IPv6 Forwarding
```dotnetcli
使客户端通过服务器的外网网卡共享上网。
sudo iptables -F
sudo iptables -P INPUT ACCEPT
sudo iptables -P OUTPUT ACCEPT
sudo iptables -P FORWARD ACCEPT
sudo iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE


重启需要重新配置
sudo ip6tables -F
sudo ip6tables -P INPUT ACCEPT
sudo ip6tables -P OUTPUT ACCEPT
sudo ip6tables -P FORWARD ACCEPT
sudo ip6tables -t nat -A POSTROUTING -o eth0 -j MASQUERADE

或者
ip6tables -t nat -A POSTROUTING -s 2001:1::/64 -j MASQUERADE
查看 ip6tables -t nat  -nvL --line-number
删除 ip6tables -t nat -D POSTROUTING 1
```

## 1.7 添加ipv6回程路由
```dotnetcli
##DHCP+native情况下，下挂终端会不通，需添加回程路由
ip -6 route add 2001:1:5::/64 via 2001:1::2000
ip -6 route add 2001:1:5::/64 via 2001:1::d635:38ff:fe77:6e88
##这个2001:1::2000是路由器wan口的获取到的地址
每次重启需要重新添加
https://mirrors.bieringer.de/Linux+IPv6-HOWTO/nat-netfilter6..html

- RFC 3633——DHCPv6 中的 IPv6 前缀选项
- RFC 4192——不设定割接日期，为一个 IPv6 网络重新分配地址的步骤
- RFC 4241——一种 IPv6/IPv4 双栈互联网接入层模型
- RFC 6177——终端站点 IPv6 地址分配（当前最佳实践）
- RFC 6879——IPv6 企业网地址重新分配的场景、注意事项和方法
- APNIC guidelines for IPv6 allocation and assignment requests——亚太地区网络信息中心 IPv6 地址分配方针
- RIPE Preparing an IPv6 Addressing Plan——准备一个 IPv6 编址计划
- 
查看规则
sudo iptables  -t  nat  -nL
cat /proc/sys/net/ipv6/conf/eth0/accept_ra
查看ipv6 路由表
ip -6 route show dev eth0
apt install iptables-persistent netfilter-persistent
netfilter-persistent save
netfilter-persistent start

iptables-save  > /etc/iptables/rules.v4
ip6tables-save > /etc/iptables/rules.v6

iptables-restore  < /etc/iptables/rules.v4
ip6tables-restore < /etc/iptables/rules.v6

systemctl stop    netfilter-persistent
systemctl start   netfilter-persistent
systemctl restart netfilter-persistent
```
添加vpn ipv4 回程路由 
```dotnetcli
ip route add 192.168.1.0/24 via 20.1.1.1
```
## 1.8 允许root用户ssh登录
```dotnetcli
先为root账户设置密码
sudo passwd root
ubuntu@ubuntu-virtual-machine:~$ sudo passwd root
输入新的 UNIX 密码：
重新输入新的 UNIX 密码：
passwd：已成功更新密码
ubuntu@ubuntu-virtual-machine:~$ su root
密码：
root@ubuntu-virtual-machine:/home/ubuntu#

sudo vim /etc/ssh/sshd_config
将第32行（vim打开后输入:32后回车）
#PermitRootLogin prohibit-password
改为
PermitRootLogin yes
sudo service ssh restart

```
一键配置脚本
```dotnetcli
#!/bin/bash
#set root password
sudo passwd root
 
#notes Document content
sudo sed -i "s/.*root quiet_success$/#&/" /etc/pam.d/gdm-autologin
sudo sed -i "s/.*root quiet_success$/#&/" /etc/pam.d/gdm-password
 
#modify profile
sudo sed -i 's/^mesg.*/tty -s \&\& mesg n \|\| true/' /root/.profile
 
#install openssh
sudo apt install openssh-server
 
#delay
sleep 1
 
#modify conf
sudo sed -i 's/^#PermitRootLogin.*/PermitRootLogin yes/' /etc/ssh/sshd_config
 
#restart server
sudo systemctl restart ssh
```
## 1.9 ubuntu 修改hostname
```dotnetcli
vi /etc/hostname
vi /etc/host
```
## 1.10 允许root用户登录桌面
```dotnetcli
1.10.1
vim /usr/share/lightdm/lightdm.conf.d/50-ubuntu.conf
添加一句
greeter-show-manual-login=true
1.10.2
vim /etc/pam.d/gdm-autologin 
注释第三行
#auth   required        pam_succeed_if.so user != root quiet_success
1.10.3
vim /etc/pam.d/gdm-password
注释该行
#auth   required        pam_succeed_if.so user != root quiet_success
1.10.4
 vim /root/.profile
~/.profile: executed by Bourne-compatible login shells.
if [ "$BASH" ]; then
 if [ -f ~/.bashrc ]; then
 . ~/.bashrc
 fi
fi
mesg n || true
_______________
修改为如下
#~/.profile: executed by Bourne-compatible login shells.
if [ “$BASH” ]; then
if [ -f ~/.bashrc ]; then
. ~/.bashrc
fi
fi
tty -s && mesg n || true
#mesg n || true

```
## 1.11 中文目录切换为英文
打开终端，在终端中输入命令: 
```dotnetcli
export LANG=en_US
xdg-user-dirs-gtk-update
跳出对话框询问是否将目录转化为英文路径,同意并关闭.
在终端中输入命令:
export LANG=zh_CN
关闭终端,并重起.下次进入系统,系统会提示是否把转化好的目录改回中文.选择不再提示,并取消修改.主目录的中文转英文就完成了
```

## 1.12 关闭休眠
```dotnetcli
查看休眠状态
sudo systemctl status sleep.target
关闭休眠
systemctl mask sleep.target suspend.target hibernate.target hybrid-sleep.target
```

## 1.13 查看电脑性能模式
```dotnetcli
cat /sys/devices/system/cpu/cpu*/cpufreq/scaling_governor
如果没有启用高性能模式一般为：powersave
apt-get install indicator-cpufreq
重启电脑
电脑右上角出现图标，可以选择节能模式还是高性能模式

安装cpu频率管理软件
apt install cpufrequtils
查看cpu状态
cpufreq-info
调整cpu到性能模式
cpufreq-set -g performance
使用上述方式,重启系统后又回到默认方式。修改默认模式: 
1、安装sysfsutils：
sudo apt-get install sysfsutils
2、查看当前的调节器：
cat /sys/devices/system/cpu/cpu0/cpufreq/scaling_governor

3、编辑/etc/sysfs.conf ,增加如下语句:
vi /sys/devices/system/cpu/cpu0/cpufreq/scaling_governor
```
## 1.14 修改欢迎信息
```dotnetcli
vi /etc/update-motd.d/00-header
run-parts /etc/update-motd.d/
```
## 1.15 清理不必要的内核文件

```dotnetcli
root@server:~# dpkg --get-selections | grep linux-image
linux-image-4.14.0-041400-generic               install
linux-image-4.15.0-190-generic                  install
linux-image-4.4.0-112-generic                   deinstall
linux-image-4.4.0-116-generic                   deinstall
linux-image-4.4.0-119-generic                   deinstall
linux-image-4.4.0-121-generic                   deinstall
linux-image-4.4.0-124-generic                   deinstall
linux-image-4.4.0-169-generic                   deinstall
linux-image-4.4.0-170-generic                   deinstall
linux-image-4.4.0-171-generic                   deinstall
linux-image-4.4.0-173-generic                   deinstall
linux-image-4.4.0-174-generic                   deinstall
linux-image-4.4.0-176-generic                   deinstall
linux-image-4.4.0-177-generic                   deinstall
linux-image-4.4.0-178-generic                   deinstall
linux-image-4.4.0-179-generic                   deinstall
linux-image-4.4.0-184-generic                   deinstall
linux-image-4.4.0-185-generic                   deinstall
linux-image-4.4.0-186-generic                   deinstall
linux-image-4.4.0-187-generic                   deinstall
linux-image-4.4.0-189-generic                   deinstall
linux-image-4.4.0-190-generic                   deinstall
linux-image-4.4.0-193-generic                   deinstall
linux-image-4.4.0-194-generic                   deinstall
linux-image-4.4.0-197-generic                   deinstall
linux-image-4.4.0-198-generic                   deinstall
linux-image-4.4.0-200-generic                   deinstall
linux-image-4.4.0-201-generic                   deinstall
linux-image-4.4.0-203-generic                   deinstall
linux-image-4.4.0-204-generic                   deinstall
linux-image-4.4.0-206-generic                   deinstall
linux-image-4.4.0-208-generic                   deinstall
linux-image-4.4.0-209-generic                   deinstall
linux-image-4.4.0-21-generic                    deinstall
linux-image-4.4.0-210-generic                   deinstall
linux-image-extra-4.4.0-112-generic             deinstall
linux-image-extra-4.4.0-116-generic             deinstall
linux-image-extra-4.4.0-119-generic             deinstall
linux-image-extra-4.4.0-121-generic             deinstall
linux-image-extra-4.4.0-124-generic             deinstall
linux-image-extra-4.4.0-21-generic              deinstall
linux-image-generic                             install
```
这些是deinstall的内核文件，是可以删除的
```dotnetcli
apt purge -y linux-image-4.4.0*
 apt purge -y linux-image-extra-4.4.0*
 
 
root@miwifi-autotest:~# dpkg --get-selections | grep linux-imag
linux-image-4.14.0-041400-generic               install
linux-image-4.15.0-190-generic                  install
linux-image-generic                             install
```

## 1.16 升级ubuntu 内核

```dotnetcli
根据需要执行
 apt-get upgrade linux-image-generic
```
手工升级
```dotnetcli
最新的稳定版是 v4.20
https://kernel.ubuntu.com/~kernel-ppa/mainline/v4.20/
找到你对应的架构（“Build for XXX”）的那部分。然后下载符合以下格式的两个文件（其中 X.Y.Z 是最高版本号）：
linux-image-X.Y.Z-generic-*.deb
linux-modules-X.Y.Z-generic-.deb
执行此命令手动安装内核：
$ sudo dpkg --install *.deb
重启系统，使用新内核：
$ sudo reboot
```

## 1.17 升级ubuntu 版本
```
do-release-upgrade  
```
## 1.18 快捷脚本及别名
```
root@server:~# cat << EOF > 4.sh && chmod 755 4.sh
chmod 777 /var/lib/dhcp/dhcpd.leases
dhcpd -4 -cf /etc/dhcp/dhcpd.conf eth1
sudo iptables -F
sudo iptables -P INPUT ACCEPT
sudo iptables -P OUTPUT ACCEPT
sudo iptables -P FORWARD ACCEPT
sudo iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
> EOF
root@ubuntu:~#
root@ubuntu:~# cat << EOF > 6.sh && chmod 755 6.sh
sudo chmod 777 /var/lib/dhcp/dhcpd6.leases
sudo dhcpd -6 -cf /etc/dhcp/dhcpd6.conf eth1
sudo ip6tables -F
sudo ip6tables -P INPUT ACCEPT
sudo ip6tables -P OUTPUT ACCEPT
sudo ip6tables -P FORWARD ACCEPT
sudo ip6tables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
>EOF
vi /root/.bashrc
修改bashrc文件，永久增加4条alias方便使用
alias accelstart='accel-pppd -d -c /etc/accel-ppp.conf'
alias kill='kill -9'
alias ps='ps -ef | grep -v grep'
alias guolv='egrep -v "^#|^$" ' 
alias lg='lazygit'
alias ra='ranger'

##命令行输出改为英文
export LANGUAGE=en_US 
export LANG=en_US.UTF-8 

source /root/.bashrc

#更改console界面输出为英文
export LC_MESSAGES=C
export LC_ALL=C
过滤掉文件中注释的行和非空行
sed -n "/^\s[^# \t].$/p"
举例
guolv /etc/apt/sources.list
```
## 1.19 设置日志文件大小
```dotnetcli
journalctl --vacuum-size=50M
超过后会自动清理
```

## 1.20 清除DNS缓存
linux本身是没有dns缓存的,想使用dns缓存的话需要自己安装一个服务程序NSCD(name service cache daemon). 
Nscd会缓存libc接口(比如 getpwnam(3), getpwuid(3), getgrnam(3), getgrgid(3), gethostbyname(3))发起的名称服务的请求。
nscd缓存三种服务passwd, group, hosts，所以它会记录三个库，分别对应源/etc/passwd, /etc/hosts 和 /etc/resolv.conf每个库保存两份缓存，一份是找到记录的，一份是没有找到记录的。每一种缓存都保存有生存时间（TTL）.
如果您已经在本地缓存了不正确的 DNS 条目，那么您需要清空您的缓存来使 DNS 客户端提出新的 DNS 请求并更新解析结果。当然，您也可以等缓存的 DNS 条目过期以后让系统自动冲掉该条目……这通常需要24个小时。
在 Ubuntu 中冲掉 DNS 缓存的方式是重新启动 nscd 守护程序。

```dotnetcli
apt install nscd
/etc/init.d/nscd restart
```
默认的配置文件是/etc/nscd.conf，通过编辑/etc/nscd.conf文件，在其中增加如下一行可以开启本地DNS cache： 
 enable-cache hosts yes 

 ## 1.21 systemctl 查看服务列表
 ```dotnetcli
 systemctl list-units --type service 
 ```

## 1.22 dpkg -i 缺少依赖
```dotnetcli
wget http://mirrors.ustc.edu.cn/debian/pool/main/a/appstream/appstream_0.14.4-1_amd64.de
root@server:~# dpkg -i appstream_0.14.4-1_amd64.deb
正在选中未选择的软件包 appstream。
(正在读取数据库 ... 系统当前共安装有 168765 个文件和目录。)
正准备解包 appstream_0.14.4-1_amd64.deb  ...
正在解包 appstream (0.14.4-1) ...
dpkg: 依赖关系问题使得 appstream 的配置工作不能继续：
 appstream 依赖于 libappstream4 (>= 0.14.4)；然而：
系统中 libappstream4:amd64 的版本为 0.12.0-3ubuntu1。
 appstream 依赖于 libglib2.0-0 (>= 2.58)；然而：
系统中 libglib2.0-0:amd64 的版本为 2.56.4-0ubuntu0.18.04.9。

dpkg: 处理软件包 appstream (--install)时出错：
 依赖关系问题 - 仍未被配置
正在处理用于 man-db (2.8.3-2ubuntu0.1) 的触发器 ...
在处理时有错误发生：
 appstream
root@server:~# apt --fix-broken install
正在读取软件包列表... 完成
正在分析软件包的依赖关系树
正在读取状态信息... 完成
正在修复依赖关系... 完成
下列软件包将被【卸载】：
  appstream
升级了 0 个软件包，新安装了 0 个软件包，要卸载 1 个软件包，有 0 个软件包未被升级。
有 1 个软件包没有被完全安装或卸载。
解压缩后将会空出 2,043 kB 的空间。
您希望继续执行吗？ [Y/n] y
(正在读取数据库 ... 系统当前共安装有 168823 个文件和目录。)
正在卸载 appstream (0.14.4-1) ...
正在处理用于 man-db (2.8.3-2ubuntu0.1) 的触发器 ...
root@server:~# 
```

## 1.23 关闭蓝牙
```dotnetcli
桌面虚拟组件
systemctl stop spice-vdagentd
systemctl disable spice-vdagentd

蓝牙
systemctl disable bluetooth.service
systemctl stop  bluetooth.service
```

# 2. 安装isc DHCP Server
##isc 可以用但官方已经停止维护,转为kea dhcp server ,参考本文第三部分
## 2.1 安装命令
```dotnetcli
ubuntu@server:/opt/accel-ppp-code/build$ sudo apt install isc-dhcp-server
https://kb.isc.org/docs/aa-01096
https://kb.isc.org/v1/docs/isc-dhcp-44-manual-pages-dhcpdconf
https://www.sixxs.net/wiki/Configuring_ISC_DHCPv6_Server
```

## 2.2 配置文件
sudo vi /etc/dhcp/dhcpd.conf
```dotnetcli
option classless-static-routes code 249 = array of unsigned integer 8;
option static-routes code 33 = array of unsigned integer 8;
subnet 192.168.1.0 netmask 255.255.255.0 {
range 192.168.1.21 192.168.1.254;
option domain-name "xiaomi.com";
option domain-name-servers 192.168.1.1, 223.5.5.5, 223.6.6.6,10.237.25.6,10.237.25.7;
option subnet-mask 255.255.255.0;
option routers 192.168.1.1;
option broadcast-address 192.168.2.255;
default-lease-time 600;
max-lease-time 7200;
}
```
sudo vi /etc/dhcp/dhcpd6.conf
```dotnetcli
authoritative;
default-lease-time 600;
max-lease-time 7200;
log-facility local7;
option dhcp6.name-servers 2001:1::1;
#option dhcp6.domain-search "domain.example";
subnet6 2001:1::/64{
    range6 2001:1::1000 2001:1::2000;
    range6 2001:1::/64 temporary;
    option dhcp6.name-servers 2001:1::1;
    prefix6 2001:1:5:0:: 2001:1:5:0:: /48; 
}
```
阿里IPv6公共DNS（2400:3200::1或2400:3200:baba::1）

## 2.3 启动命令及权限
设置权限
```dotnetcli
sudo chmod 777 /var/lib/dhcp/dhcpd.leases && sudo chmod 777 /var/lib/dhcp/dhcpd6.leases
sudo dhcpd -4 -cf /etc/dhcp/dhcpd.conf eth1 
sudo dhcpd -6 -cf /etc/dhcp/dhcpd6.conf eth1
sudo service isc-dhcp-server6 start
systemctl disable isc-dhcp-server
systemctl disable isc-dhcp-server6
```

如果不设置权限会报类似错误
```dotnetcli
ubuntu@server:~$ sudo dhcpd -6 -cf /etc/dhcp/dhcpd6.conf eth1
Internet Systems Consortium DHCP Server 4.3.5
Copyright 2004-2016 Internet Systems Consortium.
All rights reserved.
For info, please visit https://www.isc.org/software/dhcp/
Config file: /etc/dhcp/dhcpd6.conf
Database file: /var/lib/dhcp/dhcpd6.leases
PID file: /var/run/dhcpd6.pid
Can't open /var/lib/dhcp/dhcpd6.leases for append.

If you think you have received this message due to a bug rather
than a configuration issue please read the section on submitting
bugs on either our web page at www.isc.org or in the README file
before submitting a bug.  These pages explain the proper
process and the information we find helpful for debugging..

exiting.
ubuntu@server:~$
```

```dotnetcli
sudo dhcpd -4 -cf /etc/dhcp/dhcpd.conf eth1
sudo dhcpd -6 -cf /etc/dhcp/dhcpd6.conf eth1

debug模式启动
sudo dhcpd -6 -d -cf /etc/dhcp/dhcpd6.conf eth1
```

设置开机自启动
https://www.sixxs.net/wiki/Configuring_ISC_DHCPv6_Server#DHCP_Server_Configfiles_.28Create_a_Range.29

服务器查看dhcp分配情况
```dotnetcli
 cat /var/lib/dhcp/dhcpd.leases
 cat /var/lib/dhcp/dhcpd6.leases
```
## 2.5 ipv6
windows上IPv6地址的%
![Alt text](/img/image-1.png)
%是网卡索引编号，用于添加路由等
![Alt text](/img/image-2.png)
![Alt text](/img/image-3.png)
EUI-64 规范生成链路本地地址规则
  a 在接口mac地址中间部分插入FF:FE
  b 把接口mac地址从做导游第七位进行二进制翻转

windows在windows7 以后 默认的链路本地地址为随机生成，与mac地址没有任何关系
该特性相关命令
```dotnetcli
netsh interface ipv6 set global randomizeidentifiers=enable 开启随机
netsh interface ipv6 set global randomizeidentifiers=disable 关闭随机
``` 
```dotnetcli
以太网适配器 10Gb:

   连接特定的 DNS 后缀 . . . . . . . :
   IPv6 地址 . . . . . . . . . . . . : fd00:6868:6868::656
   IPv6 地址 . . . . . . . . . . . . : fd00:6868:6868:0:b51a:1c34:f530:cc09
   临时 IPv6 地址. . . . . . . . . . : fd00:6868:6868:0:f5ba:fd44:533b:e1b3
   本地链接 IPv6 地址. . . . . . . . : fe80::d7e5:b12a:8325:f25%24
   IPv4 地址 . . . . . . . . . . . . : 10.31.1.116
   子网掩码  . . . . . . . . . . . . : 255.255.0.0
   默认网关. . . . . . . . . . . . . : fe80::9e9d:7eff:fe79:ed77%24
                                       10.31.0.1
                                       
######################################################
windows 关闭随机之后获取到的地址
以太网适配器 10GB:

   连接特定的 DNS 后缀 . . . . . . . :
   描述. . . . . . . . . . . . . . . : Aquantia AQtion 10Gbit Network Adapter
   物理地址. . . . . . . . . . . . . : E0-E1-A9-10-83-D1
   DHCP 已启用 . . . . . . . . . . . : 是
   自动配置已启用. . . . . . . . . . : 是
   IPv6 地址 . . . . . . . . . . . . : fd00:6868:6868::e0e(首选)
   获得租约的时间  . . . . . . . . . : 2022年12月7日 18:49:47
   租约过期的时间  . . . . . . . . . : 2022年12月8日 6:49:47
   IPv6 地址 . . . . . . . . . . . . : fd00:6868:6868:0:e2e1:a9ff:fe10:83d1(首选)
   临时 IPv6 地址. . . . . . . . . . : fd00:6868:6868:0:35e1:f72:3eee:b258(首选)
   本地链接 IPv6 地址. . . . . . . . : fe80::e2e1:a9ff:fe10:83d1%24(首选)
   IPv4 地址 . . . . . . . . . . . . : 10.31.1.116(首选)
   子网掩码  . . . . . . . . . . . . : 255.255.0.0
   获得租约的时间  . . . . . . . . . : 2022年12月5日 16:08:19
   租约过期的时间  . . . . . . . . . : 2022年12月8日 2:37:58
   默认网关. . . . . . . . . . . . . : fe80::9e9d:7eff:fe79:ed77%24
                                       10.31.0.1
   DHCP 服务器 . . . . . . . . . . . : 10.31.0.1
   DHCPv6 IAID . . . . . . . . . . . : 987816361
   DHCPv6 客户端 DUID  . . . . . . . : 00-01-00-01-28-0F-02-A3-F0-1E-34-14-31-07
   DNS 服务器  . . . . . . . . . . . : fe80::9e9d:7eff:fe79:ed77%24
                                       10.31.0.1
                                       fe80::9e9d:7eff:fe79:ed77%24      
  ####################################################
  mac地址第7位反转 E0-E1-A9-10-83-D1
  1110 0000= E0 
  1110 0010= E2
  E0-E1-A9-FF-FE-10-83-D1=>规则a，从中间批两半，插入FFFE
  E2-E1-A9-FF-FE-10-83-D1=>规则b，mac地址第7位反转
  E2E1:A9FF:FE10:83D1=>本地链路地址
```

ISC 已宣布 ISC DHCP 的生命周期将于 2022 年底结束。
ISC将继续为现有订户提供专业支持服务，但不打算发布任何进一步的维护版本
后续使用kea dhcp server

# 3. Kea dhcp server 安装配置
```dotnetcli
kea-2-2当前稳定分支
kea-2-3当前开发分支
```
## 3.1 官方ftp和gitlab
https://ftp.isc.org/isc/kea/
https://gitlab.isc.org/isc-projects/kea
 官方文档
https://ftp.isc.org/isc/kea/2.2.0/doc/html/

```dotnetcli
源码包方式比较复杂，依赖较多，很可能由于编译不通过而安装失败，暂不推荐

# install the build environment
apt -y install automake libtool pkg-config build-essential ccache
# install the dependencies
apt -y install libboost-dev libboost-system-dev liblog4cplus-dev libssl-dev
apt -y install  gcc build-essential make libmysql++-dev openssl libssl-dev libboost-system-dev liblog4cplus-dev liblog4cplus-1.1-9 libmysqlclient-dev
#mysql和postgressql选一个就行
#sudo apt install postgresql-server-dev-all libpq-dev 
sudu apt install mysql-server mysql-client libmysqlclient-dev
systemctl enable mysql
systemctl status mysql
apt install C++11

wget https://boostorg.jfrog.io/artifactory/main/release/1.80.0/source/boost_1_80_0.tar.gz
tar -zxvf boost_1_80_0.tar.gz
cd boost_1_80_0/
./bootstrap.sh --with-libraries=all --with-toolset=gcc
./b2 toolset=gcc
./b2 install --prefix=/usr/pkg/lib

wget https://github.com/log4cplus/log4cplus/releases/download/REL_2_0_8/log4cplus-2.0.8.tar.gz
tar -zxvf log4cplus-2.0.8.tar.gz
cd log4cplus-2.0.8/
./configure
make
make install
vi /etc/profile.d/log4cpp.sh 
LD_LIBRARY_PATH=:$LD_LIBRARY_PATH:/usr/local/lib
export LD_LIBRARY_PATH

##保存好文件后，修改该文件可执行权限，在终端中输入的命令如下：
chmod a+x /etc/profile.d/log4cpp.sh
##使用命令ldconfig -v后上述配置方可生效，在终端（ctrl+alt+t）中直接输入ldconfig -v,注意需要root权限（在终端中输入sudo su，然后回车输入密码），
desktop:/usr/local/lib# ldconfig -v

git config --global http.sslVerify false
git clone https://gitlab.isc.org/isc-projects/kea.git
wget https://ftp.isc.org/isc/kea/2.2.0/kea-2.2.0.tar.gz
tar xvfz kea-2.2.0.tar.gz
cd kea-2.2.0

export PKG_CONFIG_PATH=/usr/local/lib64/pkgconfig
# export CC="ccache gcc" CXX="ccache g++"
declare -x PATH="/usr/lib64/ccache:$PATH"
autoreconf --install
#./configure --with-boost-include=/usr/pkg/include --with-mysql=/usr/bin/mysql_config --prefix=/etc/kea --enable-perfdhcp --enable-shell --with-boost-libs=-lboost_system --with-boost-lib-dir=/usr/pkg/lib

./configure --prefix=/etc/kea --enable-perfdhcp --enable-shell
make -j4
make install
echo "/usr/local/lib/hooks" > /etc/ld.so.conf.d/kea.conf
ldconfig
```
```dotnetcli
编译参数 
 ./configure使用开关运行--help以查看不同的选项。一些常用的选项是：
--prefix 定义安装位置（默认为/usr/local）。
--with-mysql 使用代码构建 Kea，使其能够在 MySQL 数据库中存储租约和主机预订。
--with-pgsql 使用代码构建 Kea，使其能够在 PostgreSQL 数据库中存储租约和主机预订。
--with-log4cplus 定义查找 Log4cplus 标头和库的路径。通常这是没有必要的。
--with-boost-include 定义查找 Boost 标头的路径。通常这是没有必要的。
--with-botan-config 指定 botan-config 脚本的路径以使用 Botan 构建用于加密功能。最好使用 OpenSSL（见下文）。
--with-openssl 使用 OpenSSL 加密库而不是 Botan。默认 configure搜索有效的 Botan 安装；如果找不到，Kea 会搜索 OpenSSL。通常这是没有必要的。
--enable-shell 构建可选kea-shell工具（更多内容在The Kea Shell中）。默认是不构建它。
--with-site-packages 只有在kea-shell启用时才有用，此开关会导致 kea-shell Python 包安装在指定目录中。这对与 Debian 相关的发行版非常有用。虽然大多数系统将 Python 包存储在 中${prefix}/usr/lib/pythonX/site-packages，但 Debian 为从 DEB 安装的包引入了一个单独的目录。预计此类 Python 包将安装在 /usr/lib/python3/dist-packages.
--enable-perfdhcp 构建可选的perfdhcpDHCP 基准测试工具。默认是不构建它。
--with-freeradius 构建可选的RADIUS挂钩。此选项指定 FreeRADIUS 客户端的修补版本的路径。此功能在 Kea 的仅限订阅者版本中可用，并且需要仅限订阅的 RADIUS 挂钩。
--with-freeradius-dictionary 为 FreeRADIUS 字典文件指定一个非标准位置，其中包含支持的 RADIUS 属性列表。此功能在 Kea 的仅限订阅者版本中可用，并且需要仅限订阅的 RADIUS 挂钩。
有许多选项通常对普通用户来说不是必需的。然而，它们可能对包维护者、开发者或想要扩展 Kea 代码或发送补丁的人有用
--with-gtest,--with-gtest-source 启用使用 Google 测试框架构建 C++ 单元测试。此选项指定 gtest 源的路径。（如果系统没有安装该框架，可以从https://github.com/google/googletest下载。）
--enable-generate-docs 启用 Kea 文档的重建。ISC 为每个版本发布 Kea 文档；然而，在某些情况下可能需要重建它：例如，更改文档中的某些内容，或者从尚未发布的 git 源生成新的文档。
--enable-generate-parser 使用 flex 或 bison 启用解析器的生成。Kea 源包括 .cc 和 .h 解析器文件，为方便用户而预先生成。默认情况下，Kea 不使用 flex 或 bison，以避免要求用户安装不必要的依赖项。但是，如果解析器中的任何内容发生更改（例如添加新参数），则需要 flex 和 bison 重新生成解析器。此选项允许这样做。
--enable-generate-messages 允许从消息源文件重新生成消息文件，例如使用 Kea 消息编译器从 xxx_messages.mes 重新生成 xxx_messages.h 和 xxx_messages.cc。默认情况下，Kea 是使用发行版中的这些 .h 和 .cc 文件构建的。但是，如果 .mes 文件中的任何内容发生更改（例如添加新消息），则需要构建和使用 Kea 消息编译器。此选项允许这样做。
```

## 3.3 通过apt源进行安装
```dotnetcli
通过apt源安装 较为简单，推荐

apt install curl
curl -1sLf   'https://dl.cloudsmith.io/public/isc/kea-2-2/setup.deb.sh'   | sudo -E bash

apt-get install isc-kea*
```

## 3.4 默认配置文件
root@ubuntu:/etc/kea# ls
kea-ctrl-agent.conf  kea-dhcp4.conf  kea-dhcp6.conf  kea-dhcp-ddns.conf

## 3.5 修改后配置文件
##kea的配置文件采用JSON格式
##配置文件没有规定必须以什么结尾，json conf都可以
##配置文件必须是格式正确的 JSON，这意味着任何给定范围的参数必须用逗号分隔，并且最后一个参数后不能有逗号。

json在线解析
https://www.bejson.com/jsoneditoronline/
dhcpv4配置手册
https://ftp.isc.org/isc/kea/2.2.0/doc/html/arm/dhcp4-srv.html

参数解释

valid-lifetime定义服务器给出的地址（租约）的有效期

renew-timer和 rebind-timer是定义 T1 和 T2 计时器的值（也以秒为单位），它们控制客户端何时开始更新和重新绑定过程
interfaces-config映射指定服务器应在其上侦听 DHCP 消息的网络接口
列表用方括号打开和关闭，元素用逗号分隔 

### 3.5.1 DHCPv4 配置文件
```dotnetcli
{
        "Dhcp4": {
                "valid-lifetime": 4000,
                "renew-timer": 1000,
                "rebind-timer": 2000,
                "interfaces-config": {
                        "interfaces": ["eth1"]
                },
                "lease-database": {
                        "type": "memfile",
                        "persist": true,
                        "name": "/var/lib/kea/dhcp4.leases"
                },
                "option-data": [{
                                "name": "domain-name-servers",
                                "data": "223.5.5.5,223.6.6.6",
                                "always-send": true
                        },
                        {
                                "name": "routers",
                                "data": "192.168.1.1",
                                "always-send": true
                        }
                ],
                "subnet4": [{
                        "subnet": "192.168.1.0/24",
                        "pools": [{
                                "pool": "192.168.1.10 - 192.168.1.100"
                        }]

                }]
        }
}
```

valid-lifetime定义服务器给出的地址（租约）的有效期；
默认情况下允许客户端使用给定地址 4000 秒。（请注意，整数按原样指定，没有任何引号。）
renew-timer并且rebind-timer是定义 T1 和 T2 计时器的值（也以秒为单位），
这些计时器控制客户端何时开始更新和重新绑定过程。
dhcpv6配置手册
https://ftp.isc.org/isc/kea/2.2.0/doc/html/arm/dhcp6-srv.html

```dotnetcli
Memfile - 租赁的基本存储
    persist：控制是否将新租约和对现有租约的更新写入文件
    name：指定记录新租约和租约更新的租约文件的绝对位置
    name 默认值为/var/lib/kea/kea-leases6.csv
    lfc-interval: 指定服务器将执行租用文件清理 (LFC) 的时间间隔（以秒为单位）
    max-row-errors：指定服务器停止尝试加载租用文件之前的行错误数,当服务器加载租约文件时，它会逐行处理，每行包含一个租约。如果一行有缺陷并且无法正确处理，服务器会记录它，丢弃该行，然后继续处理下一行。此参数可用于设置可能发生的此类丢弃的数量限制，之后服务器放弃努力并退出。默认值0禁用限制并允许服务器处理整个文件，无论丢弃多少行。

pd-pools 一个子网可能有一个或多个前缀委托池,为子网启用前缀委托
prefix 每个池都有一个前缀地址，指定为前缀 
prefix-len  前缀长度
delegated-len 委托前缀长度
委托长度不得短于（即它必须在数值上大于或等于）前缀长度
如果委托长度和前缀长度相等，服务器将只能委托一个前缀。委托前缀不必与子网前缀匹配。
 "pd-pools": [
                {
                    "prefix": "3000:1::",
                    "prefix-len": 64,
                    "delegated-len": 96
                }
            ]   
```

###  3.5.2 DHCPv6 配置文件

```dotnetcli
{
        "Dhcp6": {
                "valid-lifetime": 4000,
                "renew-timer": 1000,
                "rebind-timer": 2000,
                "preferred-lifetime": 3000,
                "interfaces-config": {
                        "interfaces": ["eth1"]
                },
                "lease-database": {
                        "type": "memfile",
                        "persist": true,
                        #"name": "/var/lib/kea/dhcp6.leases"
                },
                "option-data": [{
                        "name": "dns-servers",
                        "data": "2001:1::1,2400:3200::1,2400:3200:baba::1,2408:8899::8,2408:8888::8",
                        "always-send": true
                }],
                "subnet6": [{
                        "subnet": "2001:1::/64",
                        "pd-pools": [{
                                "prefix": "2011:1::",
                                "prefix-len": 32,
                                "delegated-len": 48
                        }],
                        #"id":10,
                        "pools": [{"pool": "2001:1::/96"}],

                 "interface" : "eth1"
                }]
        }
}
```

### 3.5.3 DHCPv6无状态配置文件
Kea 服务器支持无状态模式。客户端可以发送 Information-Request 消息，服务器返回带有请求选项的答案，前提是这些选项在服务器配置中可用。服务器首先尝试使用每个子网的选项；如果由于任何原因失败，它会尝试提供在全局范围内定义的选项。
无状态和有状态模式可以一起使用。不需要特殊的配置指令来处理这个问题；只需使用有状态客户端的配置，无状态客户端将仅获得他们请求的选项。

```dotnetcli
{
"Dhcp6": {
    "interfaces-config": {
        "interfaces": [ "eth1" ]
    },
    "option-data": [ {
        "name": "dns-servers",
        "data": "2001:1::1, 2400:3200::1"
    } ],
    "lease-database": {
        "type": "memfile"
    }
 }
```

### 3.5.4 DHCPv6 多dns配置
```dotnetcli
{
        "Dhcp6": {
                "valid-lifetime": 4000,
                "renew-timer": 1000,
                "rebind-timer": 2000,
                "preferred-lifetime": 3000,
                "interfaces-config": {
                        "interfaces": ["eth1"]
                },
                "lease-database": {
                        "type": "memfile",
                        "persist": true,
                        "name": "/var/lib/kea/dhcp6.leases"
                },
                "option-data": [{
                                "name": "dns-servers",
                                "data": "2001:1::1,2400:3200::1,2400:3200:baba::1,2402:4e00::,2400:da00::6666,240e:4c:4008::1,240e:4c:4808::1,2408:8899::8,2408:8888::8,2409:8088::a,2409:8088::b,240C::6666,240C::6644,2001:dc7:1000::1,2001:de4::101,2001:de4::102,2001:da8:202:10::36,2001:da8:202:10::37,2001:da8:8000:1:202:120:2:100,2001:da8:8000:1:202:120:2:101,2001:cc0:2fff:1::6666,2001:da8:205:2060::188,2001:da8:ff:305:20c:29ff:fe1f:a92a,2001:da8::666,2001:da8:208:10::6,2001:4860:4860::8888,2001:4860:4860::8844,2606:4700:4700::1111,2606:4700:4700::1001,2620:0:ccc::2,2620:0:ccd::2,2620:fe::fe,2620:fe::9",
                                "always-send": true
                        }

                ],
                "subnet6": [{
                        "subnet": "2001:1:3::/48",
                        "pd-pools": [{
                                "prefix": "fd00:6868:6868::",
                                "prefix-len": 48,
                                "delegated-len": 64
                        }],

                        "pools": [{
                                        "pool": "2001:1:3:1::1-2001:1:3:1::1000"
                                },
                                {
                                        "pool": "2001:1:3:2::1-2001:1:3:2::1000"
                                }
                        ],

                        "interface": "eth1"
                }]
        }
}
```

### 3.5.5 DHCPv6两步交互快速分配过程
```dotnetcli
相关rfc介绍
rfc3315#section-22.14
此项路由器不一定支持 
两步交互区别于四步交互常用于网络中只有一个DHCPv6服务器的情况。DHCPv6客户端首先通过组播发送Solicit报文来定位可以为其提供服务的DHCPv6服务器，DHCPv6服务器收到客户端的Solicit报文后，为其分配地址和配置信息，直接回应Reply报文，完成地址申请和分配过程。
两步交换可以提高DHCPv6过程的效率，但在有多个DHCPv6服务器的网络中，多个DHCPv6服务器都可以为DHCPv6客户端分配IPv6地址，回应Reply报文，但是客户端实际只可能使用其中一个服务器为其分配的IPv6地址和配置信息。为了防止这种情况的发生，管理员可以配置DHCPv6服务器是否支持两步交互地址分配方式。
- DHCPv6服务器端如果配置使能了两步交互，并且客户端报文中也包含Rapid Commit选项，服务器采用两步交互方式为客户端分配地址和配置信息。
- 如果DHCPv6服务器不支持快速分配地址，则采用四步交互方式为客户端分配IPv6地址和其他网络配置参数。

```
![Alt text](/img/image-4.png)
![Alt text](/img/image-5.png)

支持快速配置 Rapid Commit 服务器端配置文件
```dotnetcli
{
        "Dhcp6": {
                "valid-lifetime": 4000,
                "renew-timer": 1000,
                "rebind-timer": 2000,
                "preferred-lifetime": 3000,
                "interfaces-config": {
                        "interfaces": ["eth1"]
                },
                "lease-database": {
                        "type": "memfile",
                        "persist": true,
                        "name": "/var/lib/kea/dhcp6.leases"
                },
                "option-data": [{
                        "name": "dns-servers",
                        "data": "2001:1::1,2400:3200::1,2400:3200:baba::1,2408:8899::8,2408:8888::8",
                        "always-send": true
                }],
                "subnet6": [{
                        "subnet": "2001:1:3::/48",
                        "rapid-commit": true,
                         "pd-pools": [{
                                "prefix": "2001:1:3:5::",
                                "prefix-len": 32,
                                "delegated-len": 48
                        }],
                        #"id":10,
                        "pools": [{"pool": "2001:1:3:1::1-2001:1:3:1::1000"},
                                  { "pool": "2001:1:3:2::1-2001:1:3:2::1000" }],

                 "interface" : "eth1"
                }]
        }
}
```

### 3.5.6 公共IPv6 dns列表
|                       |                     |
|-----------------------|---------------------|
| 阿里 IPv6 DNS           | 2400:3200::1        |
| 阿里 IPv6 DNS           | 2400:3200:baba::1   |
| 腾讯 DNSPod IPv6        | 2402:4e00::         |
| 百度 IPv6 DNS           | 2400:da00::6666     |
| 中国电信 IPv6 DNS         | 240e:4c:4008::1     |
|                       | 240e:4c:4808::1     |
| 中国联通 IPv6 DNS         | 2408:8899::8        |
|                       | 2408:8888::8        |
| 中国移动 IPv6 DNS         | 2409:8088::a        |
|                       | 2409:8088::b        |
| 下一代互联网北京研究中心          | 240C::6666          |
|                       | 240C::6644          |
| CNNIC IPv6 DNS 服务器    | 2001:dc7:1000::1    |
|  台湾网络资讯中心提供的免费公共 DNS  | 2001:de4::101       |
|                       | 2001:de4::102       |
|  北京邮电大学 IPv6 DNS 服务器  | 2001:da8:202:10::36 |
|                       | 2001:da8:202:10::37 |

### 3.5.7 6In4 6to4 暂不涉及
  https://ftp.isc.org/isc/kea/2.2.0/doc/html/arm/dhcp6-srv.html#common-softwire46-options
IPv6 tunnel (6in4, 6to4) 

## 3.6 IPv6地址的几种配置方式
路由器的ipv6地址获取跟路由器的ipv6是native模式还是nat6模式
以及服务器的radvd的配置和DHCPv6的配置都有点关系
| 方式               |                                                                                      | 地址/前缀  | 网关     | DNS    | 备注               |
|------------------|--------------------------------------------------------------------------------------|--------|--------|--------|------------------|
| 静态               |                                                                                      | 手工     | 手工     | 手工     |                  |
| SLAAC            | Stateless Address Autoconfiguration                                                  | RS和RA  | RS和RA  | 无      | M=0, O=0         |
| Stateful DHCPv6  | 无状态地址自动配置                                                                            |        | DHCPv6 | DHCPv6 |                  |
|                  |  应用于没有DHCPv6服务器的环境。主机使用RA消息中的前缀构造IPv6单播地址，同时使用其他方法（非DHCPv6），例如手工配置的方法设置其他配置信息（DNS等）  | DHCPv6 |        |        | M=1，O=1          |
| Stateless DHCPv6 | 有状态DHCPv6                                                                            |        | RS和RA  | DHCPv6 | 常见               |
|                  |  主机使用DHCPv6来配置IPv6单播地址以及其他配置信息（DNS等）。这种应用也称为DHCPv6 Stateful。                         | RS和RA  |        |        | M=1, O=0 不常见     |
|                  | 主机仅仅使用DHCPv6来获取IPv6地址，至于其他配置信息则并不通过DHCPv6获得，这种组合不建议使用                                |        |        |        | 设置了M的标记位会忽略O的标记位 |
|                  | 无状态DHCPv6                                                                            |        |        |        | M=0, O=1         |
|                  |  主机使用RA消息获得的IPv6前缀构造IPv6地址，同时使用DHCPv6来获取除了地址之外的其他配置信息。这种应用也被称为DHCPv6 stateless       |        |        |        |                  |


下面节选了rfc4861 中关于M O标记位部分的描述
```dotnetcli
ICMP Fields:

      Type           134

      Code           0

      Checksum       The ICMP checksum.  See [ICMPv6].

      Cur Hop Limit  8-bit unsigned integer.  The default value that
                     should be placed in the Hop Count field of the IP
                     header for outgoing IP packets.  A value of zero
                     means unspecified (by this router).

      M              1-bit "Managed address configuration" flag.  When
                     set, it indicates that addresses are available via
                     Dynamic Host Configuration Protocol [DHCPv6].

                     If the M flag is set, the O flag is redundant and
                     can be ignored because DHCPv6 will return all
                     available configuration information.

      O              1-bit "Other configuration" flag.  When set, it
                     indicates that other configuration information is
                     available via DHCPv6.  Examples of such information
                     are DNS-related information or information on other
                     servers within the network.

        Note: If neither M nor O flags are set, this indicates that no
        information is available via DHCPv6.
```
### 3.6.1 SLAAC
##
如果server端只有SLAAC ，没有启用dhcpv6的话
那路由器在native模式下是无法生成lan侧ipv6地址的，这是符合预期的
SLAAC 是指当 dhcpv6 server端仅仅开启radvd，但路由器lan侧生成地址需要RA报文里面支持PD选项 （ Prefix Delegation (IA_PD) ）
而PD选项是dhcp的rfc里的选项，radvd不支持该选项
具体请参考rfc3633 中的IA_PD Prefix option
那radvd的M=0 O=0时，开启DHCPv6, 配置采用stateless，路由器在native模式下是可以获取到lan侧ipv6地址的
![Alt text](/img/image-6.png)
路由器nat6模式下为
![Alt text](/img/image-7.png)
没有DNS 是因为radvd配置中缺少RDNSS,配置之后效果如
![Alt text](/img/image-8.png)
此时，路由器获取的默认网关为RA的源IP地址，也就是服务器的link-local地址 
服务器地址如下图
![Alt text](/img/image-9.png)
路由器默认路由表
![Alt text](/img/image-10.png)
此时radvd配置如下
```dotnetcli
interface eth1 {
      AdvSendAdvert on;
      MinRtrAdvInterval 10;
      MaxRtrAdvInterval 20;

AdvSourceLLAddress on;
AdvManagedFlag off;
AdvOtherConfigFlag off;
AdvDefaultLifetime 1800;
AdvReachableTime 0;
AdvRetransTimer 0;
AdvSourceLLAddress off;

prefix 2001:1::/64{
                AdvOnLink on;
                AdvAutonomous on;
                AdvRouterAddr on;
                AdvValidLifetime 600;
                AdvPreferredLifetime 600;
                };

route 2001:1::/64
           {
           };
RDNSS 2400:3200::1
          {
                AdvRDNSSPreference 8;
                AdvRDNSSLifetime 180;
          };
};
```

Dhcpv6未开启，配置略
抓包查看RA 报文M O 标记位均为0 
![Alt text](/img/image-11.png)

### 3.6.7 ICMPv6 目标不可达消息 

```dotnetcli
https://www.rfc-editor.org/rfc/rfc4443#section-3.1

 Code             0 - No route to destination
                  1 - Communication with destination
                        administratively prohibited
                  2 - Beyond scope of source address
                  3 - Address unreachable
                  4 - Port unreachable
                  5 - Source address failed ingress/egress policy
                  6 - Reject route to destination
```
```dotnetcli
关于发送 ICMPv6 目标不可达消息
设备向源发送以下 ICMPv6 目标不可达消息：
ICMPv6 No Route to Destination 消息— 要转发的数据包不匹配任何路由。
ICMPv6 Communication with Destination Administratively Prohibited 消息- 行政禁止正在阻止与目的地的成功通信。这通常是由设备上的防火墙或 ACL 引起的。
ICMPv6 Beyond Scope of Source Address 消息——目的地超出了源 IPv6 地址的范围。例如，数据包的源 IPv6 地址是链路本地地址，其目标 IPv6 地址是全球单播地址。
ICMPv6 Address Unreachable 消息— 设备无法解析数据包目标 IPv6 地址的链路层地址。
ICMPv6 Port Unreachable 消息— 接收到的 UDP 数据包在目标设备上不存在端口进程。
```
### 3.6.8 windows  EUI-64 规范生成链路本地地址规则
```dotnetcli
windows在windows7 以后 默认的链路本地地址为随机生成，与mac地址没有任何关系

可以使用下面的命令关闭windows随机生成mac地址的方式

netsh interface ipv6 set global randomizeidentifiers=enable 开启随机
netsh interface ipv6 set global randomizeidentifiers=disable 关闭随机
```
```dotnetcli
关闭之后pc获取到的ipv6地址和mac存在关联关系
以太网适配器 以太网 2:

   连接特定的 DNS 后缀 . . . . . . . :
   描述. . . . . . . . . . . . . . . : Realtek PCIe GbE Family Controller
   物理地址. . . . . . . . . . . . . : 60-18-95-2B-C7-BC
   DHCP 已启用 . . . . . . . . . . . : 是
   自动配置已启用. . . . . . . . . . : 是
   IPv6 地址 . . . . . . . . . . . . : 2011:1:3:19::456(首选)
   获得租约的时间  . . . . . . . . . : 2022年12月28日 19:10:25
   租约过期的时间  . . . . . . . . . : 2022年12月28日 20:00:43
   IPv6 地址 . . . . . . . . . . . . : 2011:1:3:19:6218:95ff:fe2b:c7bc(首选)
   临时 IPv6 地址. . . . . . . . . . : 2011:1:3:19:6548:287:e69d:9a27(首选)
   临时 IPv6 地址. . . . . . . . . . : 2011:1:3:19:80b7:481c:3190:c7f2(首选)
   本地链接 IPv6 地址. . . . . . . . : fe80::6218:95ff:fe2b:c7bc%17(首选)
   IPv4 地址 . . . . . . . . . . . . : 192.168.31.236(首选)
   子网掩码  . . . . . . . . . . . . : 255.255.255.0
   获得租约的时间  . . . . . . . . . : 2022年12月28日 19:10:33
   租约过期的时间  . . . . . . . . . : 2022年12月29日 7:10:33
   默认网关. . . . . . . . . . . . . : fe80::8e53:c3ff:fe40:634%17
                                       192.168.31.1
   DHCP 服务器 . . . . . . . . . . . : 192.168.31.1
   DHCPv6 IAID . . . . . . . . . . . : 241178773
   DHCPv6 客户端 DUID  . . . . . . . : 00-01-00-01-29-0C-00-DC-60-18-95-2B-C7-BC
   DNS 服务器  . . . . . . . . . . . : fe80::8e53:c3ff:fe40:634%17
                                       192.168.31.1
   TCPIP 上的 NetBIOS  . . . . . . . : 已启用
```

## 3.7 日志文件位置
```dotnetcli
/var/log/kea-dhcp4.log
/var/log/kea-dhcp6.log
/var/log/kea-ddns.log
/var/log/kea-ctrl-agent.log
```

## 3.8 启动命令
```dotnetcli
systemctl start  isc-kea-dhcp4-server.service
systemctl enable  isc-kea-dhcp4-server.service

systemctl start  isc-kea-dhcp6-server.service
systemctl enable  isc-kea-dhcp6-server.service 


systemctl stop isc-kea-dhcp4-server.service
systemctl disable  isc-kea-dhcp4-server.service
systemctl stop isc-kea-dhcp6-server.service
systemctl disable isc-kea-dhcp6-server.service
systemctl stop isc-kea-dhcp6-server.service
systemctl disable isc-kea-dhcp6-server.service
systemctl stop isc-kea-dhcp-ddns-server.service
systemctl disable isc-kea-dhcp-ddns-server.service
systemctl stop isc-kea-ctrl-agent.service
systemctl disable isc-kea-ctrl-agent.service

kea-dhcp4 -c /etc/kea/v4.conf
kea-dhcp6 -c /etc/kea/v6.conf

cd /etc/kea
kea-dhcp6 -c v6.conf -d 以debug模式启动，若频繁修改配置文件，可以用-d参数

##重启服务
systemctl restart  isc-kea-dhcp4-server.service
systemctl restart  isc-kea-dhcp6-server.service 
```

## 3.9 V4 启动脚本
```dotnetcli
kea-dhcp4 -c /etc/kea/v4.conf
sudo iptables -F
sudo iptables -P INPUT ACCEPT
sudo iptables -P OUTPUT ACCEPT
sudo iptables -P FORWARD ACCEPT
sudo iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
```

## 3.10 v6 启动脚本

```dotnetcli
kea-dhcp6 -c /etc/kea/v6.conf
sudo ip6tables -F
sudo ip6tables -P INPUT ACCEPT
sudo ip6tables -P OUTPUT ACCEPT
sudo ip6tables -P FORWARD ACCEPT
sudo ip6tables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
```
Perfdhcp 官方的dhcp压力测试工具
https://www.systutorials.com/docs/linux/man/8-perfdhcp/

4.  安装RADVD
路由广播守护（The Router Advertisement Daemon，简称：radvd）是一个符合RFC 2461使用邻居发现协议用于实现IPv6地址本地链接广播和IPv6路由前缀的开源软件。[2]该软件是给系统管理员用于实现在IPv6下对主机进行无状态自动配置地址。
当主机配置其网络接口时，会向网络多播一个路由请求来发现可用的路由，radvd会响应一个路由广播（router advertisement，RA）的消息。此外，radvd会定期在网络多播RA包给连接其的链接来更新主机信息。RA包包含了该链接的路由前缀，最大传输单元（MTU），默认响应的路由器地址等信息。
radvd也支持发布符合RFC 6106要求的可选信息，有递归DNS地址、DNS搜索列表等
IPv6 has a lot more support for autoconfiguration than IPv4. But for this autoconfiguration to work on the hosts of a network, the routers of the local network have to run a program which answers the autoconfiguration requests of the hosts. . On Linux this program is called radvd, which stands for Router ADVertisement Daemon. This daemon listens to router solicitations (RS) and answers with router advertisement (RA). Furthermore unsolicited RAs are also sent from time to time.

对于无状态自动配置的ipv6测试需要搭建radvd服务器，这样在路由器/设备发送RS请求(icmpv6 type133)的时候，radvd服务器就可以返回RA消息(icmpv6 type134)，告诉设备全局地址的前缀，设备自己再结合接口ID算出一个可聚集全局单播地址。
由于IPv6的 Router Advertisement 无状态自动配置 stateless在目前的标准下[1]只能告知客户端此网段的ipv6 prefix和default gateway(网关的linklocal地址)，因此，如果要实现更加详细的资讯配置，只能使用RADVD+DHCPv6进行协同工作，即进行DHCPv6的stateful配置。
但是，由于DHCPv6不能告知客户端默认路由，默认路由的广播只能靠RA，这样就必须在RA报文里面，不报告“A”（自动配置），只报告“R”（路由前缀），让客户端通过DHCPv6去获取默认路由
更多请参考项目主页
https://radvd.litech.org/
http://manpages.ubuntu.com/manpages/trusty/man8/radvd.8.html
https://manpages.debian.org/testing/radvd/radvd.conf.5.en.html
https://linux.die.net/man/5/radvd.conf
https://github.com/radvd-project/radvd/blob/master/radvd.conf.example
https://github.com/radvd-project/radvd

## 4.1 安装命令
```dotnetcli
sudo apt install radvd
```

## 4.3 配置文件
```dotnetcli
AdvSendAdvert 表示允许广播Router Advertisement和Router       Solicitation消息。
AdvManagedFlag 表示禁止有状态自动配置。(RFC2462)
 #主机使用管理(有状态)协议进行地址自动配置,以及使用无状态地址自动配置自动配置的任何地址。默认关闭(off)
#此项参数代表是否开启有状态配置 即标志位M bit值为1时代表on开启 为0时代表off关闭 
AdvOtherConfigFlag 表示禁止自动配置其他信息。 (RFC2462)
 #主机使用管理(有状态)协议自动配置其他(非地址)信息。
 #此项参数代表是否使用DHCP分配除地址以外的参数,例如DNS服务器等 即标志位O bit值为1时代表on开启 为0时代表off关闭 
```
```dotnetcli
sudo vi /etc/radvd.conf
interface eth1 {
      AdvSendAdvert on;
      MinRtrAdvInterval 10;
      MaxRtrAdvInterval 30;

AdvSourceLLAddress on;
AdvManagedFlag on;
AdvOtherConfigFlag on;
AdvDefaultLifetime 1800;
AdvReachableTime 0;
AdvRetransTimer 0;
AdvSourceLLAddress off;

prefix 2001:1::/64{
                AdvOnLink on;
                AdvAutonomous on;
                AdvRouterAddr on;
                AdvValidLifetime 600;
                AdvPreferredLifetime 600;
                };
route 2001:1::1/64{
                };
                
RDNSS 2400:3200::1 240c::6666
          {
                AdvRDNSSPreference 8;
                AdvRDNSSLifetime 30;
          };

};
```

http://manpages.ubuntu.com/manpages/trusty/man5/radvd.conf.5.html
配置文件说明
```dotnetcli
interface eth1
{
AdvSendAdvert on;           #启用路由器公告（RA）功能
MinRtrAdvInterval 30;          #每隔30-100秒间隔发送公告消息
MaxRtrAdvInterval 100;
#spf
AdvManagedFlag on;          # M值
AdvOtherConfigFlag on;          # O 值
#spf
prefix 2001:db8:1:0::/64            #发送的前缀信息
{
AdvOnLink on;
AdvAutonomous on;   #公告的前缀可用来自动位置配置
AdvRouterAddr off;
};
#DNS部分未经过测试
#RDNSS
#RDNS 2001:db8:1:0::1   #提供RA的DNS选项，目前支持RFC6106支持的普遍性不高
#例如WIN7尚未支持RFC6106，所以并不会取得RA的DNS选项
#这部分就需要通过DHCPv6来解决
#   {
#       AdvRDNSSPreference 8;
#       AdvRDNSSLifetime 180;
#   };
#
};
```

| IgnoreIfMissing off         | 是否忽略这个网口配置(例如网口不存在) 默认关闭(off)                                                                                                     |
|-----------------------------|-----------------------------------------------------------------------------------------------------------------------------------|
| AdvSendAdvert off           | 是否(开启)路由通告转发                                                                                                                      |
| UnicastOnly off             | 接口类型仅支持单播(会阻止未经请求的转发),对于非广播多路访问链接(例如ISATAP),必须开启                                                                                  |
| MaxRtrAdvInterval 600       | 从接口发送未经请求的多播路由器播发之间允许的最大时间,默认600秒                                                                                                 |
| MinRtrAdvInterval 99        | 从接口发送未经请求的多播路由器播发之间的最短时间,默认0.33**MaxRtrAdvInterval                                                                                |
| MinDelayBetweenRAs 3        | 从接口发送多播路由器播发之间的最短时间 默认3秒                                                                                                          |
| AdvManagedFlag off          | 主机使用管理(有状态)协议进行地址自动配置,以及使用无状态地址自动配置自动配置的任何地址。默认关闭(off)                                                                            |
| AdvOtherConfigFlag off      | 此项参数代表是否开启有状态配置                                                                                                                   |
|                             | 即标志位M bit值为1时代表on开启 为0时代表off关闭                                                                                                    |
| AdvLinkMTU 0                | 主机使用管理(有状态)协议自动配置其他(非地址)信息。                                                                                                       |
| AdvOtherConfigFlag off      | #此项参数代表是否使用DHCP分配除地址以外的参数,例如DNS服务器等                                                                                               |
| AdvLinkMTU 0                |  off 不获取dns，on获取dns                                                                                                               |
| AdvReachableTime 0          |  即标志位O bit值为1时代表on开启 为0时代表off关闭                                                                                                   |
| AdvRetransTimer             | MTU 选项用于路由器播发消息,以确保链路上的所有节点在链路 MTU 不为人知的情况下使用相同的 MTU 值 默认值0                                                                       |
| AdvCurHopLimit 64           | #如果指定,即不是 0,则不得小于 1280,并且不大于此链路允许的最大 MTU(例如以太网的最大 MTU 为 1500)                                                                     |
| AdvDefaultLifetime 1800     | 主机使用管理(有状态)协议自动配置其他(非地址)信息。                                                                                                       |
| AdvDefaultPreference medium | 此项参数代表是否使用DHCP分配除地址以外的参数,例如DNS服务器等 即标志位O bit值为1时代表on开启 为0时代表off关闭                                                                 |
| AdvSourceLLAddress on       | MTU 选项用于路由器播发消息,以确保链路上的所有节点在链路 MTU 不为人知的情况下使用相同的 MTU 值 默认值0                                                                       |
| AdvHomeAgentFlag off        | 如果指定,即不是 0,则不得小于 1280,并且不大于此链路允许的最大 MTU(例如以太网的最大 MTU 为 1500)                                                                      |
| AdvHomeAgentInfo off        | 节点假定邻居在收到可到达性确认后可到达的时间(以毫秒为单位)。由邻居不可移动检测算法使用。值为零表示未指定                                                                             |
| HomeAgentLifetime 1800      | 重新传输邻居请求消息之间的时间（以毫秒为单位）。由地址解析和邻居无法访问检测算法使用。值为零表示未指定                                                                               |
| HomeAgentPreference 0       | 应放置在传出（单播）IP 数据包的 IP 标头的跃点计数字段中的默认值。该值应设置为 Internet 的当前直径。值零表示未指定                                                                 |
| AdvMobRtrSupportFlag off    | 与默认路由器关联的生存期最大值对应于 18.2 小时。生存期为 0 表示路由器不是默认路由器,不应出现在默认路由器列表中。路由器生存期仅适用于路由器作为默认路由器的有用性;                                            |
| AdvIntervalOpt off          | 与默认路由器关联的首选项，如"low", "medium", or "high" 默认 medium                                                                                |
|                             | 设置后，传出接口的链接层地址包含在 RA 中 默认 on                                                                                                      |
| AdvOnLink on                | 设置后，指示发送路由器能够充当移动 IPv6 家庭代理。设置后，移动 IPv6 指定的最小限制用于 MinRtrAdvInterval 和 MaxRtrAdvInterval                                           |
| AdvAutonomous on            | 设置后，路由器通告中包含主代理信息选项（由移动 IPv6 指定）。使用此选项时，还必须设置 AdvHomeAgentFlag                                                                    |
| AdvRouterAddr off;          | 家庭代理服务的时间长度                                                                                                                       |
| AdvValidLifetime 2592000    | 送此路由器播发的主页代理的首选项                                                                                                                  |
| AdvPreferredLifetime 604800 | 设置后，主代理会发出信号，支持移动路由器注册（由 NEMO Basic 指定）。使用此选项时，还必须设置 AdvHomeAgentInfo                                                             |
| Base6to4Interface 6to4      | 设置后，路由器播发中包含播发间隔选项                                                                                                                |
| DeprecatePrefix on          |                                                                                                                                   |
| DecrementLifetimes on|off   | 设置后，指示可用于链路上确定前缀                                                                                                                  |
|                             | 设置后，指示前缀可用于 RFC 2462 中指定的自治地址配置                                                                                                   |
|                             | 此项参数代表是否开启无状态配置(radvd分配) 即标志位A bit值为1时代表on开启 为0时代表off关闭 默认值 on                                                                    |
| AdvRouteLifetime 1800;      | 发送接口地址而不是网络前缀                                                                                                                     |
| AdvRoutePreference medium   | 前缀对于链路上确定有效的时间长度（相对于数据包发送时间）。 默认值 2592000秒 30天                                                                                    |
|                             | 通过无状态地址自动配置从前缀生成的地址的时间长度 以秒为单位 默认值604800 7天                                                                                       |
|                             | 如果指定此选项，则前缀将与接口名称的 IPv4 地址结合，以生成有效的 6to4 前缀前缀的前 16 位将被 2002 替换，后 32 位前缀将被配置时分配给接口名称的 IPv4 地址替换。前缀的剩余 80位（包括SLAID）将根据需要配置文件中指定进行播发 |

## 4.4 启动命令
```dotnetcli
sudo systemctl enable radvd
 sudo systemctl start radvd

ubuntu@ubuntu-virtual-machine:~$ sudo systemctl start radvd
ubuntu@ubuntu-virtual-machine:~$ ps | grep radvd
ubuntu@ubuntu-virtual-machine:~$ ps -ee | grep radvd
 31639 ?        00:00:00 radvd
 31640 ?        00:00:00 radvd
ubuntu@ubuntu-virtual-machine:~$
sudo systemctl status radvd.service
```

## 5. 安装npd6

Neighbor Proxy Daemon IPv6 
 为网关路由设备接收的IPv6邻居请求提供代理服务的Linux守护程序。 
参考 https://manpages.ubuntu.com/manpages/bionic/man5/npd6.conf.5.html
https://github.com/npd6/npd6

安装命令
apt-get install npd6

5.3 配置文件
```dotnetcli
root@ubuntu:/etc/init.d# vi /etc/npd6.conf

 # General parameters
        prefix = 2001:1::/64
        interface=eth0
  #      listtype  = none
  #      linkOption =  false
  #      ignoreLocal  =  true
  #      routerNA =  true
  #      maxHops =  255
  #     pollErrorLimit = 20
```
这里填外网网卡

# 6. 安装accel-ppp server
参考官方文档，步骤比较详细，一步步照着做就行了
https://accel-ppp.readthedocs.io/en/latest/installation/ubuntu.html

```dotnetcli
sudo apt-get install -y build-essential cmake gcc linux-headers-`uname -r` git libpcre3-dev libssl-dev liblua5.1-0-dev
sudo git clone https://github.com/accel-ppp/accel-ppp.git /opt/accel-ppp-code
sudo mkdir /opt/accel-ppp-code/build
sudo cd /opt/accel-ppp-code/build/
cmake -DBUILD_IPOE_DRIVER=TRUE -DBUILD_VLAN_MON_DRIVER=TRUE -DCMAKE_INSTALL_PREFIX=/usr -DKDIR=/usr/src/linux-headers-uname -r-DLUA=TRUE -DCPACK_TYPE=Ubuntu18 ..
sudo cmake -DBUILD_IPOE_DRIVER=TRUE -DBUILD_VLAN_MON_DRIVER=TRUE -DCMAKE_INSTALL_PREFIX=/usr -DKDIR=/usr/src/linux-headers-uname -r -DLUA=TRUE -DCPACK_TYPE=Ubuntu18 ..
sudo make
sudo cpack -G DEB
sudo dpkg -i accel-ppp.deb
sudo mv /etc/accel-ppp.conf.dist /etc/accel-ppp.conf
sudo systemctl start accel-ppp
sudo accel-pppd -d -c /etc/accel-ppp.conf -p /var/run/accel-ppp.pid
```

## 6.3 实际配置文件
实际的配置文件 
sudo vi /etc/accel-ppp.conf
```dotnetcli
[modules]
log_file
#log_syslog
#log_tcp
#log_pgsql

pptp
l2tp
#sstp
pppoe
#ipoe

auth_mschap_v2
auth_mschap_v1
auth_chap_md5
auth_pap

#radius
chap-secrets
pap-secrets
ippool

pppd_compat

#shaper
#net-snmp
#logwtmp
#connlimit

ipv6_nd
ipv6_dhcp
ipv6pool

[core]
log-error=/var/log/accel-ppp/core.log
thread-count=8

[common]
#single-session=replace
#single-session-ignore-case=0
#sid-case=upper
#sid-source=seq
#max-sessions=1000
#max-starting=0
#check-ip=0

[ppp]
verbose=1
min-mtu=1280
mtu=1492
mru=1492
#accomp=deny
#pcomp=deny
#ccp=0
#mppe=require
ipv4=require
ipv6=allow
ipv6-intf-id=0:0:0:1
ipv6-peer-intf-id=ipv4
ipv6-accept-peer-intf-id=1
lcp-echo-interval=20
#lcp-echo-failure=3
lcp-echo-timeout=120
unit-cache=1
#unit-preallocate=1

[auth]
#any-login=0
#noauth=0

[pptp]
verbose=1
bind=20.1.1.1
#echo-interval=30
ip-pool=pptp
mtu=1460
mru=1460
ppp-max-mtu=1460
mppe=128
#mppe=require
mppe=allow
#ipv6-pool=pptp
#ipv6-pool-delegate=pptp
#ifname=pptp%d

[pppoe]
verbose=1
#ac-name=xxx
#service-name=yyy
#pado-delay=0
#pado-delay=0,100:100,200:200,-1:500
called-sid=mac
#tr101=1
#padi-limit=0
ip-pool=pppoe
ipv6-pool=pppoev6
ipv6-pool-delegate=pppoedele
#ifname=pppoe%d
#sid-uppercase=0
#vlan-mon=eth0,10-200
#vlan-timeout=60
#vlan-name=%I.%N
#interface=eth1,padi-limit=1000
interface=eth1

[l2tp]
verbose=1
bind=20.1.1.1
#dictionary=/usr/local/share/accel-ppp/l2tp/dictionary
hello-interval=20
timeout=10
#rtimeout=1
#rtimeout-cap=16
retransmit=3
ppp-max-mtu=1460
#recv-window=16
#host-name=accel-ppp
#dir300_quirk=0
#secret=
#dataseq=allow
#reorder-timeout=0
ip-pool=l2tp
#ipv6-pool=l2tp
#ipv6-pool-delegate=l2tp
#ifname=l2tp%d

[sstp]
verbose=1
#cert-hash-proto=sha1,sha256
#cert-hash-sha1=
#cert-hash-sha256=
#accept=ssl,proxy
#ssl-protocol=tls1,tls1.1,tls1.2,tls1.3
#ssl-dhparam=/etc/ssl/dhparam.pem
#ssl-ecdh-curve=prime256v1
#ssl-ciphers=DEFAULT
#ssl-prefer-server-ciphers=0
#ssl-ca-file=/etc/ssl/sstp-ca.crt
#ssl-pemfile=/etc/ssl/sstp-cert.pem
#ssl-keyfile=/etc/ssl/sstp-key.pem
#host-name=domain.tld
#http-error=allow
#timeout=60
#hello-interval=60
#ip-pool=sstp
#ipv6-pool=sstp
#ipv6-pool-delegate=sstp
#ifname=sstp%d

[ipoe]
verbose=1
username=ifname
#password=username
lease-time=600
#renew-time=300
#rebind-time=525
max-lease-time=3600
#unit-cache=1000
#l4-redirect-table=4
#l4-redirect-ipset=l4
#l4-redirect-on-reject=300
#l4-redirect-ip-pool=pool1
shared=0
ifcfg=1
mode=L2
start=dhcpv4
#start=up
#ip-unnumbered=1
#proxy-arp=0
#nat=0
#proto=100
#relay=10.10.10.10
#vendor=Custom
#weight=0
#attr-dhcp-client-ip=DHCP-Client-IP-Address
#attr-dhcp-router-ip=DHCP-Router-IP-Address
#attr-dhcp-mask=DHCP-Mask
#attr-dhcp-lease-time=DHCP-Lease-Time
#attr-dhcp-renew-time=DHCP-Renewal-Time
#attr-dhcp-rebind-time=DHCP-Rebinding-Time
#attr-dhcp-opt82=DHCP-Option82
#attr-dhcp-opt82-remote-id=DHCP-Agent-Remote-Id
#attr-dhcp-opt82-circuit-id=DHCP-Agent-Circuit-Id
#attr-l4-redirect=L4-Redirect
#attr-l4-redirect-table=4
#attr-l4-redirect-ipset=l4-redirect
#lua-file=/etc/accel-ppp.lua
#offer-delay=0,100:100,200:200,-1:1000
#vlan-mon=eth0,10-200
#vlan-timeout=60
#vlan-name=%I.%N
#ip-pool=ipoe
#ipv6-pool=ipoe
#ipv6-pool-delegate=ipoe
#idle-timeout=0
#session-timeout=0
#soft-terminate=0
#check-mac-change=1
#calling-sid=mac
#local-net=192.168.0.0/16
interface=eth1

[dns]
dns1=223.5.5.5
#dns2=223.6.6.6

[wins]
#wins1=172.16.0.1
#wins2=172.16.1.1

[radius]
#dictionary=/usr/local/share/accel-ppp/radius/dictionary
nas-identifier=accel-ppp
nas-ip-address=127.0.0.1
gw-ip-address=192.168.100.1
server=127.0.0.1,testing123,auth-port=1812,acct-port=1813,req-limit=50,fail-timeout=0,max-fail=10,weight=1
dae-server=127.0.0.1:3799,testing123
verbose=1
#timeout=3
#max-try=3
#acct-timeout=120
#acct-delay-time=0
#acct-on=0
#acct-interim-interval=0
#acct-interim-jitter=0
#default-realm=
#strip-realm=0
#attr-tunnel-type=My-Tunnel-Type

[client-ip-range]
#10.0.0.0/8
disable

[ip-pool]
gw-ip-address=192.168.1.1
#vendor=Cisco
#attr=Cisco-AVPair
attr=Framed-Pool
#192.168.0.2-255
192.168.1.2-255,name=pppoe
20.1.1.2-255,name=pptp
20.1.1.2-255,name=l2tp
#192.168.4.2-255,name=pool4,next=pool1
#192.168.4.0/24

[log]
log-file=/var/log/accel-ppp/accel-ppp.log
log-emerg=/var/log/accel-ppp/emerg.log
log-fail-file=/var/log/accel-ppp/auth-fail.log
#log-debug=/dev/stdout
#syslog=accel-pppd,daemon
#log-tcp=127.0.0.1:3000
copy=1
#color=1
#per-user-dir=per_user
#per-session-dir=per_session
#per-session=1
level=3

[log-pgsql]
conninfo=user=log
log-table=log

[pppd-compat]
verbose=1
#ip-pre-up=/etc/ppp/ip-pre-up
ip-up=/etc/ppp/ip-up
ip-down=/etc/ppp/ip-down
#ip-change=/etc/ppp/ip-change
radattr-prefix=/var/run/radattr
#fork-limit=16

[chap-secrets]
#gw-ip-address=192.168.100.1
chap-secrets=/etc/ppp/chap-secrets
#encrypted=0
#username-hash=md5
[pap-secrets]
pap-secrets=/etc/ppp/pap-secrets

[shaper]
#attr=Filter-Id
#down-burst-factor=0.1
#up-burst-factor=1.0
#latency=50
#mpu=0
#mtu=0
#r2q=10
#quantum=1500
#moderate-quantum=1
#cburst=1534
#ifb=ifb0
up-limiter=police
down-limiter=tbf
#leaf-qdisc=sfq perturb 10
#leaf-qdisc=fq_codel [limit PACKETS] [flows NUMBER] [target TIME] [interval TIME] [quantum BYTES] [[no]ecn]
#rate-multiplier=1
#fwmark=1
#rate-limit=2048/1024
verbose=1

[cli]
verbose=1
telnet=127.0.0.1:2000
tcp=127.0.0.1:2001
#password=123
#sessions-columns=ifname,username,ip,ip6,ip6-dp,type,state,uptime,uptime-raw,calling-sid,called-sid,sid,comp,rx-bytes,tx-bytes,rx-bytes-raw,tx-bytes-raw,rx-pkts,tx-pkts

[snmp]
master=0
agent-name=accel-ppp

[connlimit]
limit=10/min
burst=3
timeout=60

[ipv6-pool]
gw-ip6-address=2001:1::1
#gw-ip6-address=fc00:0:1::1
#fc00:0:1::/48,64
#vendor=
#attr-prefix=Delegated-IPv6-Prefix-Pool
#attr-address=Stateful-IPv6-Address-Pool
2001:1:2::/48,64,name=pppoev6
##RA 报文相关,子网超过64会导致路由器获取不到地址
#fc00:0:3::/48,64,name=pool2,next=pool1
delegate=2001:1:3::/36,48,name=pppoedele
##前缀委托，子网数不宜超过64, 即路由器web上lan侧获取的ip地址
#delegate=fc00:3::/36,48,name=pool4,next=pool3

[ipv6-dns]
2400:3200::1
#5::6
#fc00:1::1
#fc00:1::2
#fc00:1::3
#dnssl=suffix1.local.net
#dnssl=suffix2.local.net.

#[ipv6-pool]
#vendor=vendor
#attr-prefix=2001:5

[ipv6-dhcp]
verbose=1
pref-lifetime=604800
valid-lifetime=2592000
route-via-gw=1

[ipv6-nd]
verbose=1
AdvManagedFlag=1
AdvOtherConfigFlag=1
AdvOnLinkFlag=1
#AdvRouterAddr=1
AdvAutonomousFlag=1

```

```dotnetcli
##文件末尾新增ipv6-nd 简单解释一下，
[ipv6-nd]
verbose=1
AdvManagedFlag=1
AdvOtherConfigFlag=1
DHCP模式下的ra报文依赖radvd，但pppoe模式下的ra报文控制，依赖ipv6-nd这几行配置文件，ipv6-nd是accel-ppp的一个模块了，功能相当于radvd，

1. 如果系统里的radvd还是开着，在开启accel-ppp ，这时候路由器会收到的两份RA，故测试pppoe的ipv6时需要关闭系统的radvd服务
2. 关闭系统的radvd服务，如果accel-ppp的配置文件没有ipv6-nd 相关的配置，则RA中的M O 位全部置1
3. 关闭系统的radvd服务，如果accel-ppp的配置文件有ipv6-nd相关配置，则RA中的M O 位置位情况根据配置文件实际值决定

```

```dotnetcli
Router Advertisement Prefix option carries three flags:
L - On-link Flag [rfc4861]
A - Autonomous Address Configuration Flag [rfc4861]
R - Router Address Flag [rfc3775 7.2部分]
Beside some refactoring to be done (the flags are not defined properly), there is a major issue: they are not fully honored.
L flag this is completely ignored - and it is a major issue (see https://tools.ietf.org/html/rfc5942 for an explanation on how much this is bad).
A flag this is checked by receiving nodes - if not set, the node will not build an address. This is honored.
R flag this is not considered either - minor problem since there's no Mobile IPv6 in the code (yet).
Moreover (if this wouldn't be bad enough), the prefix length is not honored either - meaning that any address will have a /64 prefix.
直连标记（on-link flag）。当取值为1时，表示该前缀可以作为on-link判断；否则表示该前缀不用作on-link判断，前缀本身也不包含on-link或on-off属性，默认值为1。
自动配置标记（Autonomous Address-configuration Flag）。当取值为1时，表示该前缀用于无状态地址配置；否则为有状态地址配置。默认值为1。
路由器地址标记（Router Address Flag）。用于移动IPv6（RFC3775），当取值为1时，表示Prefix字段不仅包含前缀信息，同时也包含了发送该RA报文的路由器地址。
```
### 6.4.1 创建pppoe拨号用户名和密码
https://accel-ppp.readthedocs.io/en/latest/configuration/chap_secrets.html#chap-secrets-file-example
![Alt text](/img/image-12.png)

```dotnetcli
sudo vi /etc/ppp/chap-secrets
test * test *

```

这时候可以把路由器切换为PPPoE模式拨号试试
如果拨号有错误，参阅此错误代码，大部分原因是etc/accel-ppp.conf 配置不当的问题
https://wiki.n.miui.com/pages/viewpage.action?pageId=631742813

## 6.5 启动命令
```dotnetcli
sudo accel-pppd -d -c /etc/accel-ppp.conf
或者
sudo systemctl start accel-ppp

停止命令
sudo killall  accel-pppd

重新载入配置文件
accel-cmd reload 
```
![Alt text](/img/image-13.png)
![Alt text](/img/image-14.png)

## 6.7 accel-ppp配置文件中参数具体解释参考该链接
https://github.com/accel-ppp/accel-ppp-docs/blob/master/configuration/

## 6.8 accel-ppp 日志
```dotnetcli
cat /var/log/accel-ppp/accel-ppp.log
 
 accel-cmd show stat
 
###实时查看处理器内核
perf top
apt install sysstat
sar -u ALL -P ALL 1
```

# 7. 安装bind9
目前想要支持域名vpn的话，服务器需要安装dns
(也可以临时修改host文件作为替代)
Ubuntu附带了BIND (Berkley Internet Naming Daemon)，BIND是用于在Linux上维护名称服务器的最常用程序
项目主页
https://github.com/isc-projects/bind9
 
 sudo apt install bind9
 dnsutils 软件包是测试和解决 DNS 问题非常有用的。 这些工具通常已经安装，但是要检查或安装 dnsutils，请输入以下内容
 sudo apt install dnsutils
## 7.2 配置过程
sudo vi /etc/bind/named.conf.options
```dotnetcli
options {
        directory "/var/cache/bind";
        auth-nxdomain no;    # conform to RFC1035
     // listen-on-v6 { any; };
        listen-on port 53 { localhost; 192.168.1.0/24; };
        allow-query { localhost; 192.168.1.0/24; };
        forwarders { 8.8.8.8; };
        recursion yes;
        };
``` 

sudo vi /etc/bind/named.conf.local
```dotnetcli
zone    "digger.local"   {
        type master;
        file    "/etc/bind/forward.digger.local";
 };

zone   "1.168.192.in-addr.arpa"        {
       type master;
       file    "/etc/bind/reverse.digger.local";
 };
```

sudo vi /etc/bind/forward.digger.local
```dotnetcli
$TTL    604800

@       IN      SOA     primary.digger.local. root.primary.digger.local. (
                              6         ; Serial
                         604820         ; Refresh
                          86600         ; Retry
                        2419600         ; Expire
                         604600 )       ; Negative Cache TTL

;Name Server Information
@       IN      NS      primary.digger.local.

;IP address of Your Domain Name Server(DNS)
primary IN       A      192.168.1.1

;Mail Server MX (Mail exchanger) Record
digger.local. IN  MX  10  mail.digger.local.

;A Record for Host names
www     IN       A       192.168.1.1
mail    IN       A       192.168.1.1

;CNAME Record
ftp     IN      CNAME    www.digger.local.
```

sudo vi /etc/bind/reverse.digger.local
```dotnetcli
$TTL    604800
@       IN      SOA     digger.local. root.digger.local. (
                             21         ; Serial
                         604820         ; Refresh
                          864500        ; Retry
                        2419270         ; Expire
                         604880 )       ; Negative Cache TTL

;Your Name Server Info
@       IN      NS      primary.digger.local.
primary IN      A       192.168.1.1

;Reverse Lookup for Your DNS Server
1      IN      PTR     primary.digger.local.

;PTR Record IP address to HostName
1      IN      PTR     www.digger.local.
1      IN      PTR     mail.digger.local.
```
sudo vi /etc/resolv.conf
```dotnetcli
search digger.local
nameserver 192.168.1.1
```
重启后/etc/resolv.conf添加内容消失
vi /etc/network/interfaces
dns-nameservers 192.168.1.1

## 7.3 检查配置的命令
```dotnetcli
sudo named-checkconf /etc/bind/named.conf.local
sudo named-checkzone digger.local /etc/bind/forward.digger.local
sudo named-checkzone digger.local /etc/bind/reverse.digger.local
dig primary.digger.local
nslookup www.digger.local
nslookup mail.digger.local
nslookup primary.digger.local
nslookup ftp.digger.local
```

## 7.4 启动命令
```dotnetcli
sudo systemctl start bind9.service
sudo systemctl enable bind9.service
##清除dns缓存
systemd-resolve --flush-caches 
```

# 8. wireshark

8.1 非root用户
安装以后，打开软件后，在选择网络接口进行抓包时会提示没有权限，为此，可以通过以下方法解决
    The capture session could not be initiated on interface 'eth1' (You don't have permission to capture on that device)

```dotnetcli
# 添加用户组，命名为wireshark
ubuntu@ubuntu-virtual-machine:/etc$ sudo groupadd wireshark
# 将dumpcap更改为刚添加的用户组
ubuntu@ubuntu-virtual-machine:/etc$ sudo chgrp wireshark /usr/bin/dumpcap
# 为wireshark用户组添加使用dumpcap的root权限
ubuntu@ubuntu-virtual-machine:/etc$ sudo chmod 4755 /usr/bin/dumpcap
# 将自己的用户(我的是ubuntu)添加到wireshark用户组
ubuntu@ubuntu-virtual-machine:~$ whoami
ubuntu
ubuntu@ubuntu-virtual-machine:~$
ubuntu@ubuntu-virtual-machine:/etc$ sudo gpasswd -a ubuntu wireshark
正在将用户“ubuntu”加入到“wireshark”组中
ubuntu@ubuntu-virtual-machine:/etc$
执行完成以后便可以使用wireshark正常抓包了
```    
## 8.2 root用户 
 
 root 用户打开wireshark提示
 ![Alt text](/img/image-15.png)

 ```dotnetcli
 vi /usr/share/wireshark/init.lua  使用文本编辑器打开。
更改disable_lua = false为disable_lua = true
 ```
## 9. 组播路由

https://oldwiki.archive.openwrt.org/doc/howto/udp_multicast?s[]=igmpproxy
https://forum.videolan.org/viewtopic.php?t=5018
组播源：可以通过wget方式将组播源导入到虚拟机上，组播源权限应为普通用户权限，root权限无法播放
sudo ip route add 224.0.0.0/4 dev eth1
sudo ip -6 route add FF15::1/32 dev eth1
rtp://@[ff15::1]:5004

### 9.1  如何让组播路由重启生效
ubuntu18.04 不再使用initd管理系统，改用systemd.
然而systemd改变太大，跟之前的完全不同。
```dotnetcli
vi /etc/rc.local
      chmod 0755 /etc/rc.local
```

```dotnetcli
#!/bin/sh
#
# rc.local
#
# This script is executed at the end of each multiuser runlevel.
# Make sure that the script will "exit 0" on success or any other
# value on error.
#
# In order to enable or disable this script just change the execution
# bits.
#
# By default this script does nothing.
##重启后开启accel，建议用systemctl控制accel-ppp
#accel-pppd -d -c /etc/accel-ppp.conf

##重启之后todesk 无法使用的问题，此处增加开启重启todesk进程
##是否需要删除todesk配置根据具体情况确定
##rm /opt/todesk/config/todeskd.conf
systemctl restart todeskd.service

##重启后增加组播路由
ip route add -net 224.0.0.0/4 dev eth1
ip -6 route add FF15::1/32 dev eth1
##重启后启动转发
/root/ff.sh

exit 0   
```
```dotnetcli
root@server:~# cat ff.sh
sudo iptables -F
sudo iptables -P INPUT ACCEPT
sudo iptables -P OUTPUT ACCEPT
sudo iptables -P FORWARD ACCEPT
sudo iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE

sudo ip6tables -F
sudo ip6tables -P INPUT ACCEPT
sudo ip6tables -P OUTPUT ACCEPT
sudo ip6tables -P FORWARD ACCEPT
sudo ip6tables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
```
chmod 755 /etc/rc.local 
systemd 默认会读取 /etc/systemd/system 下的配置文件，该目录下的文件会链接 /lib/systemd/system/ 下的文件。一般系统安装完 /lib/systemd/system/ 下会有 rc-local.service 文件，即我们需要的配置文件
```dotnetcli
ln -fs /lib/systemd/system/rc-local.service /etc/systemd/system/rc-local.service

vi /etc/systemd/system/rc-local.service
```

```dotnetcli
ubuntu 18.04:
sudo vi  /lib/systemd/system/rc.local.service
sudo ln -s /lib/systemd/system/rc.local.service /etc/systemd/system/rc.local.service       
ubuntu 20.04:
sudo vi /lib/systemd/system/rc-local.service  
sudo ln -s /lib/systemd/system/rc-local.service /etc/systemd/system/rc-local.service    
        
```

一般正常的启动文件主要分成三部分
 [Unit] 段: 启动顺序与依赖关系
 [Service] 段: 启动行为,如何启动，启动类型
 [Install] 段: 定义如何安装这个配置文件，即怎样做到开机启动
 可以看出，/etc/rc.local 的启动顺序是在网络后面，但是显然它少了 Install 段，也就没有定义如何做到开机启动，所以显然这样配置是无效的。 因此我们就需要在后面帮他加上 [Install] 段:
 ```dotnetcli
 sudo echo "
[Install]
WantedBy=multi-user.target
Alias=rc-local.service
" >> /etc/systemd/system/rc-local.service
 ```

```dotnetcli
#  SPDX-License-Identifier: LGPL-2.1+
#
#  This file is part of systemd.
#
#  systemd is free software; you can redistribute it and/or modify it
#  under the terms of the GNU Lesser General Public License as published by
#  the Free Software Foundation; either version 2.1 of the License, or
#  (at your option) any later version.

# This unit gets pulled automatically into multi-user.target by
# systemd-rc-local-generator if /etc/rc.local is executable.
[Unit]
Description=/etc/rc.local Compatibility
Documentation=man:systemd-rc-local-generator(8)
ConditionFileIsExecutable=/etc/rc.local
After=network.target

[Service]
Type=forking
ExecStart=/etc/rc.local start
TimeoutSec=0
RemainAfterExit=yes
GuessMainPID=no

[Install]
WantedBy=multi-user.target
Alias=rc-local.service
```

```dotnetcli
systemctl start rc-local.service
systemctl enable rc-local.service
```

## 9.2 VLC root用户运行
```dotnetcli
# 首先，运行如下命令以修改 VLC 二进制程序：
cp /usr/bin/vlc /usr/bin/vlc.backup
    sed -i 's/geteuid/getppid/' /usr/bin/vlc

# 然后，运行 vlc 命令启动即可：
vlc
RTP/MPEG Transport Stream 
226.1.2.3
不激活转码

```
ttl设置在 access output 
![Alt text](/img/image-16.png)

## 9.3 windows 多网卡终端点播不了的情况排查
我这台笔记本偶而就点播不了了,有线无线都不行, 
用netsh interface ipv4 show joins 命令查看后
发现目的组播地址226.1.1.3被加到了Loopback Pseudo-Interface接口
禁用其他网卡,重启后,恢复正常
--该现象一般多发生在多网卡的机器上
![Alt text](/img/image-17.png)

## 9.4 chariot 测试组播
![Alt text](/img/image-18.png)

## 10. Speedtest server

```dotnetcli
sudo apt install apache2 php
git clone https://github.com/librespeed/speedtest.git
cd speedtest
sudo cp -R backend example-singleServer-pretty.html *.js /var/www/html/
cd /var/www/html
sudo mv example-singleServer-pretty.html index.html
sudo chown -R www-data *
```
![Alt text](/img/image-19.png)
```dotnetcli
ubuntu 18.04 升级22.04之后apache2启动失败
错误详细信息如下
server systemd[1]: Starting The Apache HTTP Server...
server apachectl[13257]: apache2: Syntax error on line 146 of /etc/apache2/apache2.conf: Syntax error on l>
server apachectl[13254]: Action 'start' failed.
问题链接 https://stackoverflow.com/questions/72056814/apache2-wont-start-after-upgrade-to-ubuntu-22-04-lts-cannot-load-usr-lib-apa
解决办法:
a2dismod php7.4
a2enmod php8.1
a2disconf php7.4-fpm
a2enconf php8.1-fpm
systemctl reload apache2
systemctl restart apache2
```
10.1 Speedtest 测试参数说明
可修改/var/www/html/speedtest_worker.js文件进行调整，修改即使生效，不需要重启服务，日常可以修改下载上传时间为4-5小时（注意单位是秒），当压力测试了



待补充

## 10.6 中国科学技术大学测速网站
 https://test.ustc.edu.cn/

## 10.9 优化网卡收发包性能
```dotnetcli
root@server:~# ethtool -g eth3
Ring parameters for eth3:
Pre-set maximums:
RX:             4096
RX Mini:        0
RX Jumbo:       0
TX:             4096
Current hardware settings:
RX:             512
RX Mini:        0
RX Jumbo:       0
TX:             512
Pre-set maximums 中的 RX/TX 值为该网卡的 Buffer size 最大值；
Current hardware settings 中 RX/TX 值代表该网卡当前的 Buffer size 大小。
所以，设置的 Current hardware settings 的 RX/TX 值必须在 Pre-set maximums 的限制之内

ethtool -G eth0 rx 4096 tx 4096
##添加该命令到rc.local,重启自动生效
```

# 11 endpoint端
11. endpoint 端
tar zxvf pelinux_amd64_100.tar.gz
chmod 777 endpoint.install
./endpoint.install accept_license
cd /usr/local/Ixia
chmod 777 endpoint
./endpoint 
![Alt text](/img/image-20.png)
过wan的ixchariot 流量，endpoint 应在服务器侧，控制端在lan侧pc，切换方向需改脚本红圈处

nload eth4 -m 可实时查看网卡流量

# 12 离线密码

待补充

# 13 key文件

待补充

# 14 iperf3

 https://github.com/sio/router-throughput-test
或者直接apt install iperf3

```dotnetcli
 #server 
 iperf3 -s 
 
 #client 
 iperf3 -c 192.168.1.1 -t 60 -P -R
```
| 选项 | 全写                  | 描述                                                                                                                                          |
|----|---------------------|---------------------------------------------------------------------------------------------------------------------------------------------|
| -f | -format [kmgKMG]    | format to report ：kbits，Mbits，KBytes，MBytes                                                                                                 |
| -i | –interval #         | 报告格式有：k，m，g，KB，M，G                                                                                                                          |
| -p | - port #            | 单位换算：8 bit=1 Byte                                                                                                                           |
| -F | file name           | 1024 Byte=1 KB                                                                                                                              |
| -A | -affinity n/n,m     | 1024 KB=1 MB                                                                                                                                |
| -B | -bind               | 1024 MB=1 GB                                                                                                                                |
| -V | -verbose            | seconds between periodic bandwidth reports                                                                                                  |
| -J | -json               | 定期带宽报告之间的秒数                                                                                                                                 |
|    | –logfile f          | 设置每次报告之间的时间间隔，单位为秒。如果设置为非零值，就会按照此时间间隔输出测试报告。默认值为零。                                                                                          |
| -d | -debug              | server port to listen on/connect to                                                                                                         |
| -h | - help              | 侦听/连接到的服务器端口                                                                                                                                |
| -v | -version            | 设置端口，与服务器端的监听端口一致。默认是5001端口，与tcp的一样。                                                                                                        |
|    |                     | xmit/recv the specified file                                                                                                                |
| -s | - server            | set CPU affinity                                                                                                                            |
| -D | - daemon            | bind to a specific interface                                                                                                                |
| -I | –pidfile file       | 绑定到特定接口                                                                                                                                     |
| -1 | -one-off            | 绑定到主机的多个地址中的一个。对于客户端来说，这个参数设置了出栈接口。对于服务器端来说，这个参数设置入栈接口。这个参数只用于具有多网络接口的主机。在Iperf的UDP模式下，此参数用于绑定和加入一个多播组。使用范围在224.0.0.0至239.255.255.255的多播地址。 |
|    |                     | more detailed output                                                                                                                        |
| -c | - client            | output in JSON format                                                                                                                       |
| -u | - udp               | send output to a log file                                                                                                                   |
| -b | –bandwidth #[KM]    | emit debugging output                                                                                                                       |
| -t | - time #            | Show help message and quit                                                                                                                  |
| -n | - bytes #[KMG]      | 显示帮助消息并退出                                                                                                                                   |
| -r | - tradeoff往复测试模式    | show version information and quit                                                                                                           |
| -b | –blockcount #[KMG]  | 显示版本信息并退出                                                                                                                                   |
| -l | –len #[KM]          | Server specific特用于服务器                                                                                                                       |
|    | –cport              | run in server mode在服务器模式下运行                                                                                                                 |
| -P | - parallel #        | run the server as a daemon运行服务器作为后台进程                                                                                                       |
| -R | -reverse            | write PID file                                                                                                                              |
| -w | - window #[KMG]     | handle one client connection then exit                                                                                                      |
| -C | -congestion         | Client specific特用于客户端                                                                                                                       |
| -M | - set-mss #         | run in client mode, connecting to 在客户端模式下运行，连接到                                                                                             |
| -N | - no-delay          | 如果Iperf运行在服务器模式，并且用-c参数指定一个主机，那么Iperf将只接受指定主机的连接。此参数不能工作于UDP模式。                                                                             |
| -4 | -version4           | use UDP rather than TCP                                                                                                                     |
| -6 | -version6           | 使用UDP而不是TCP                                                                                                                                 |
| -L | -flowlabel N        | target bandwidth in bits/sec (0 for unlimited)                                                                                              |
| -S | -tos N s            | 目标带宽（以位/秒为单位）（0表示无限制）                                                                                                                       |
| -Z | -zerocopy           | UDP模式使用的带宽，单位bits/sec。此选项与-u选项相关。默认值是1 Mbit/sec。                                                                                            |
| -O | -omit N             | omit the first n seconds                                                                                                                    |
| -T | -title str          | prefix every output line with this string                                                                                                   |
| -： | –get-server-output  | get results from server                                                                                                                     |
|    | –udp-counters-64bit | use 64-bit counters in UDP test packets                                                                                                     |
|    |                     |                                                                                                                                             |


# 15. todesk for ubuntu 安装

##用于远程
http://hellodesk.cn/linux.html
https://www.todesk.com/linux.html

```dotnetcli
sudo systemctl stop todeskd.service
sudo systemctl start todeskd.service
sudo systemctl restart todeskd.service

vi /opt/todesk/config/config.ini
##添加这两句将密码设置为123456 
AuthMode=2
authPassEx=b0fb4df3f1b957186d4768e3ae0b7b2f0905f136b486bf1b1605c3c410cd9bbe83b5f75cb1a89ee97fc858fe047025e122ea7c1067b8    
```

