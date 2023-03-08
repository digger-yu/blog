<!-- keywords:install dperf;ubuntu; -->
<!-- description:how to install dperf ubuntu -->
<!-- coverimage:![cover](cover.jpg) -->

[1. 项目简介](#1-项目简介)

- [1. 项目简介](#1-项目简介)
  - [名词介绍](#名词介绍)
- [2. 环境说明](#2-环境说明)
- [3. dperf 能干什么](#3-dperf-能干什么)
- [4. 系统设置前提](#4-系统设置前提)
  - [4.1 修改网卡名](#41-修改网卡名)
  - [4.2 允许 root 用户 ssh 登录](#42-允许-root-用户-ssh-登录)
  - [4.3 ubuntu 修改 hostname](#43-ubuntu-修改-hostname)
  - [4.4 允许 root 用户登录桌面](#44-允许-root-用户登录桌面)
  - [4.5 NUMA](#45-numa)
- [5. 设置大页内存](#5-设置大页内存)
  - [5.1 2M 大页面设置](#51-2m-大页面设置)
  - [5.2 1G 大页面设置](#52-1g-大页面设置)
  - [5.3 vmware 虚拟机环境](#53-vmware-虚拟机环境)
  - [5.4 查看配置的大页](#54-查看配置的大页)
  - [5.5 Hugepage 名次解释](#55-hugepage-名次解释)
- [6. 升级系统 python](#6-升级系统-python)
- [7. DPDK 手工下载与安装编译](#7-dpdk-手工下载与安装编译)
  - [7.2 编译](#72-编译)
  - [7.3 验证 dpdk 安装](#73-验证-dpdk-安装)
- [ubuntu install script](#ubuntu-install-script)
- [8. dperf 下载与安装编译](#8-dperf-下载与安装编译)
  - [8.1 如何验证 dperf 成功启动](#81-如何验证-dperf-成功启动)
- [9. DPDK 绑定网卡](#9-dpdk-绑定网卡)
  - [9.2 查看 dpdk 是否支持该网卡](#92-查看-dpdk-是否支持该网卡)
  - [9.3 DPDK 支持的网卡列表](#93-dpdk-支持的网卡列表)
  - [9.4 查看网卡编号](#94-查看网卡编号)
  - [9.5 下载安装网卡 igb\_uio 驱动](#95-下载安装网卡-igb_uio-驱动)
  - [9.6 用 igb\_uio 绑定网卡](#96-用-igb_uio-绑定网卡)
  - [9.7 查看绑定结果](#97-查看绑定结果)
  - [9.8 使用参数“wc\_activate=1”加载 igb\_uio 以提高性能。](#98-使用参数wc_activate1加载-igb_uio-以提高性能)
  - [9.9 其他网卡本例不涉及](#99-其他网卡本例不涉及)
- [10 重启之后注意事项](#10-重启之后注意事项)
- [11 KNI 部分【本例不涉及】](#11-kni-部分本例不涉及)
- [12 实战部分--配置相关](#12-实战部分--配置相关)
  - [12.1 名次解释](#121-名次解释)
  - [12.2 配置关键参数](#122-配置关键参数)
  - [12.3 配置参数中的必须项](#123-配置参数中的必须项)
  - [12.4 全部参数表](#124-全部参数表)
  - [12.5 给网关配置 mac 地址](#125-给网关配置-mac-地址)
  - [12.6 FDIR](#126-fdir)
  - [12.7 L3L4RSS](#127-l3l4rss)
- [13 实战部分-测试](#13-实战部分-测试)
  - [13.1 Throughput](#131-throughput)
  - [13.2 CPS](#132-cps)
  - [13.3 CC](#133-cc)
  - [13.4 udp pps](#134-udp-pps)
  - [13.5 ipv6 cps](#135-ipv6-cps)
  - [13.7 其他宣称支持，但本文暂不涉及的项(欢迎补充)](#137-其他宣称支持但本文暂不涉及的项欢迎补充)
- [14. dperf 统计信息说明](#14-dperf-统计信息说明)
  - [关键统计信息](#关键统计信息)
  - [全部统计信息](#全部统计信息)
- [15 dperf 启动报错信息排查](#15-dperf-启动报错信息排查)
  - [其他可能遇到的错误,欢迎补充](#其他可能遇到的错误欢迎补充)

# 1. 项目简介

|            | 参考链接                                                                                                      |
| ---------- | ------------------------------------------------------------------------------------------------------------- |
| 项目地址   | https://github.com/baidu/                                                                                     |
| 实现原理   | https://github.com/baidu/dperf/blob/main/docs/design-CN.md                                                    |
| 参考链接   | [How to set up baidu dperf](https://metonymical.hatenablog.com/entry/2022/02/11/234927)                       |
|            | [dperf 快速上手](https://zhuanlan.zhihu.com/p/451340043)                                                      |
|            | [releases notes](https://github.com/baidu/dperf/releases)                                                     |
|            | [dperf FAQ](https://zhuanlan.zhihu.com/p/561093951)                                                           |
|            | [作者的知乎文章列表](https://www.zhihu.com/people/artnowben/posts?page=1)                                     |
|            | [DPVS v1.9.2 Performance Tests](https://github.com/iqiyi/dpvs/blob/master/test/release/v1.9.2/performance.md) |
|            | https://doc.dpdk.org/guides/linux_gsg/sys_reqs.html#compilation-of-the-dpdk                                   |
|            | https://doc.dpdk.org/guides-22.11/linux_gsg/nic_perf_intel_platform.html                                      |
|            | http://doc.dpdk.org/guides/nics/overview.html                                                                 |
|            | https://en.wikipedia.org/wiki/Data-rate_units                                                                 |
| dperf 作者 | pengjianzhang@gmail.com                                                                                       |

本文从新手小白的角度，记录了 dperf 从 0 开始到放弃的过程

## 名词介绍

建议了解的内容
| Label | Meaning | Explanation |
|--------------|-------------------------|------------------------------------------------------------------------|
| CPS | connections per second | The number of L4 connections established per second. |
| TPS | transactions per second | The number of L5-L7 transactions completed per second. |
| RPS | requests per second | The number of L5-L7 requests satisfied per second, often interchangeable with TPS. |
| RPC | requests per connection | The number of L5-L7 requests completed for a single L4 connection. Example: HTTP/1.1 with 100 transactions per TCP connection would be 100-RPC. |
| bps or bit/s | bits per second | The network bandwidth in bits per second. Some people use GB/s, Gb/s and Gbps interchangeably, but the unit is decidedly important. 1 GB/s (gigabyte per second) is 8 Gbps (gigabits per second) See: data-rate units. Things get a bit pedantic with GiB/s vs GB/s, but 8:1 bytes:bits is correct when working from the same base unit. |
| fps | frames per second | A measure of L2 frames per second, often used for core switching or firewalls. The distinction is that not all network frames contain valid data packets or datagrams. |
| pps | packets per second | A measure of the L3/L4 packets per second, typically for TCP and UDP traffic. |
| CC | concurrent connections | The number of concurrently established L4 connections. Example: if you have 100 CPS, and each connection has a lifetime of 100 s, then the resultant CC is 10,000 (connection rate \* connection lifetime). |

| Label     | Meaning                                       | Methodology                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                     |
| --------- | --------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| CPS       | connections per second                        | Determine the maximum number of fully functional (establishment, single small transaction, graceful termination) connections that can be established per second. Each transaction is a small object such that it will not result in a full size network frame.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                  |
| TPS / RPS | transactions per second / requests per second | Determine the maximum number of complete transactions that can be completed over a small number of long-lived connections. Each transaction requests a small object such that it will not result in a full size network frame.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                  |
| bps       | bits per second                               | Determine the maximum sustained throughput for a workload using long-lived connections and large (128 kB or higher) object sizes. This is often expressed for L4 and L7, and the numbers are typically different.Note that ADC industry metrics use a "single count" method for measuring throughput, such that traffic between the client and server is counted, but not the traffic between the client and ADC or between the ADC and the server.                                                                                                                                                                                                                                                                                                                                                                                                                                                                             |
| fps       | frames per second                             | Determine the maximum rate at which network frames can be inspected, routed, or mitigated without unintended drops. This is particularly useful when comparing network defense performance, such as TCP SYN flood mitigation.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                   |
| CC        | concurrent connections                        | Determine the maximum number of concurrent connections that can be established without forcefully terminating any of the connections. This is typically tested by establishing millions of connections over a long period of time, and leaving them in an established state. The test is valid only if the connections can successfully complete one transaction before a graceful termination. Stability is achieved when the new connection rate matches the connection termination rate due to old connections being satisfied.Example: 10,000 CPS with a 300 second wait time and 1-RPC for a 128 byte object will result in a moving plateau of 3.0M CC. The test should run for a minimum of 300 seconds after the initial connections have been established. These tests take a long time, as there are three phases - establishment (300 seconds), wait and transaction (300 seconds), graceful shutdown (300 seconds). |

# 2. 环境说明

Ubuntu 18.04 最小安装,从 0 开始

```
root@server:~# cat /proc/cpuinfo | grep 'model name' |uniq
model name      : Intel(R) Core(TM) i7-6700 CPU @ 3.40GHz
root@server:~# cat /proc/cpuinfo | grep 'cpu cores' |uniq
cpu cores       : 4
root@server:~#
root@server:~# cat /proc/meminfo | grep MemTotal
MemTotal:       16286624 kB
root@server:~#
```

# 3. dperf 能干什么

dperf 是一种基于 DPDK 的 100Gbps 网络性能和负载测试软件。
dperf 是一款基于 DPDK 的高性能 HTTP 负载测试工具,特别适用于以下测试

1. ThroughPut
2. CPS（Connection per seconds）、
3. CC（Concurrent Connection）
4. PPS (Packet Per Second)
5. ...其他不一一列举了

# 4. 系统设置前提

Ubuntu 18.04 最小安装，设置网卡名，允许 root 用户登录 ssh 桌面
内核版本 >= 4.4
glibc >= 2.7

这个 ubuntu 18.04 默认情况下已经满足要求了
<mark>下面几项根据实际需要修改</mark>

## 4.1 修改网卡名

```
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

## 4.2 允许 root 用户 ssh 登录

```
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
按i进入编辑状态，并在前加“#”注释掉“PermitRootLogin without-password”
然后加入PermitRootLogin yes
sudo service ssh restart
```

## 4.3 ubuntu 修改 hostname

```
vi /etc/hostname
vi /etc/host
```

## 4.4 允许 root 用户登录桌面

```
4.4.1
vim /usr/share/lightdm/lightdm.conf.d/50-ubuntu.conf
添加一句
greeter-show-manual-login=true
4.4.2
vim /etc/pam.d/gdm-autologin
注释该行
#auth   required        pam_succeed_if.so user != root quiet_success
4.4.3
vim /etc/pam.d/gdm-password
注释该行
#auth   required        pam_succeed_if.so user != root quiet_success
4.4.4
 vim /root/.profile
```

```
#~/.profile: executed by Bourne-compatible login shells.
if [ “$BASH” ]; then
if [ -f ~/.bashrc ]; then
. ~/.bashrc
fi
fi
tty -s && mesg n || true
#mesg n || true
重启系统
```

## 4.5 NUMA

物理网卡和与 DPDK 接口相连虚拟机都有对应的 NUMA node,
目前 dperf 只支持在不同的 NUMA node 上用不同的网卡，
如果一台物理机只有一个 NUMA node, 但有俩网卡这种情况 dperf 可能是不支持的
（这里准确的讲作者表示 dperf 是没有 NUMA 的限制，实测上配置正确的情况下可以支持的）

```
root@server:~# numactl --hardware
available: 1 nodes (0)
node 0 cpus: 0 1 2 3 4 5 6 7
node 0 size: 15904 MB
node 0 free: 333 MB
node distances:
node   0
  0:  10
root@server:~#
root@server:~# numactl -H
available: 1 nodes (0)
node 0 cpus: 0 1 2 3 4 5 6 7
node 0 size: 15904 MB
node 0 free: 10275 MB
node distances:
node   0
  0:  10
root@server:~#
root@server:~# ls /sys/devices/system/node/
has_cpu  has_memory  has_normal_memory  node0  online  possible  power  uevent
root@server:~#
node0下保存着相关的信息
root@server:~# ls /sys/devices/system/node/node0



root@server:~# dmidecode -t memory | grep Locator
        Locator: DIMM1
        Bank Locator: Not Specified
        Locator: DIMM2
        Bank Locator: Not Specified
        Locator: DIMM3
        Bank Locator: Not Specified
        Locator: DIMM4
        Bank Locator: Not Specified
root@server:~# dmidecode -t memory | grep Speed
        Speed: 2133 MT/s
        Configured Clock Speed: 2133 MT/s
        Speed: 2133 MT/s
        Configured Clock Speed: 2133 MT/s
        Speed: Unknown
        Configured Clock Speed: Unknown
        Speed: Unknown
        Configured Clock Speed: Unknown
root@server:~# lspci -s 05:00.0 -vv | grep LnkSta
pcilib: sysfs_read_vpd: read failed: Input/output error
                LnkSta: Speed 5GT/s, Width x4, TrErr- Train- SlotClk+ DLActive- BWMgmt- ABWMgmt-
                LnkSta2: Current De-emphasis Level: -6dB, EqualizationComplete-, EqualizationPhase1-
root@server:~#
```

# 5. 设置大页内存

<mark>dperf 运行之前必须配置大页</mark>
作者表示：如果你的系统支持 1G，<mark>最好配置为 1G 大页</mark>

大页内存相关知识点较多，此处不展开
[官方设置参考文档](http://dpdk-guide.gitlab.io/dpdk-guide/setup/hugepages.html)

```
root@dperf:~# free -g
              总计         已用        空闲      共享    缓冲/缓存    可用
内存：           3           0           0        0         2         2
交换：           1           0           1
root@dperf:~#
通过"free -g"命令查看系统有多少大页，从上面可以看到系统有3G内存。
推荐设置的大页数为系统内存的一半，如果dperf报告内存不够，再增加大页即可。

注意大多数 x86_64 系统支持各种大小的大页面。
通过运行以下命令进行检查系统是否支持1GB大页面
root@server:~# if grep pdpe1gb /proc/cpuinfo >/dev/null 2>&1; then echo "1GB supported."; fi
1GB supported.
root@server:~#


```

## 5.1 2M 大页面设置

```
DPDK 应用程序通常需要大页面支持，出于加速目的，这应该足以让它运行
Fedora 和 RHEL 默认为 2M 大页面：
sysctl -w vm.nr_hugepages=204800

对于现实世界的应用程序，您希望使此类分配永久化。您可以通过将其添加到目录/etc/sysctl.conf或目录中的单独文件来执行此操作/etc/sysctl.d/：
echo 'vm.nr_hugepages=2048' > /etc/sysctl.d/hugepages.conf

如果您尝试运行 Pktgen 的主机具有 NUMA，即节点 0 和节点 1，
则必须按照DPDK 入门指南中所述在两个 NUMA 节点上配置大页面，即
echo 2048 > /sys/devices/system/node/node0/hugepages/hugepages-2048kB/nr_hugepages
echo 2048 > /sys/devices/system/node/node1/hugepages/hugepages-2048kB/nr_hugepages
```

## 5.2 1G 大页面设置

```
//要将其永久添加到内核命令行，
//请将其附加到 /etc/default/grub 中的 GRUB_CMDLINE_LINUX，
vi /etc/default/grub
// 添加如下内容
GRUB_CMDLINE_LINUX="net.ifnames=0 biosdevname=0 default_hugepagesz=1GB hugepagesz=1G hugepages=4"
上面例子：设置每个大页1G，设置4个大页。
// 更新 grub 配置
sudo update-grub
// 设置后重启生效
reboot
```

说明

```
#grub-mkconfig是liunx通用的用来配置grub.cfg 的命令，
#update-grub是ubuntu特有的用来配置grub.cfg 的命令。
#root@server:~/dperf# cat /usr/sbin/update-grub
##!/bin/sh
#set -e
#exec grub-mkconfig -o /boot/grub/grub.cfg "$@"
#root@server:~/dperf#
#update-grub实际上也是调用的grub-mkconfig
#一般用update-grub就可以了
```

## 5.3 vmware 虚拟机环境

```
#vi /boot/grub/grub.cfg
#default_hugepagesz = 1 GB  hugepagesz = 1 G  hugepages = 4
#linux16 /vmlinuz-xxx ...   transparent_hugepage=never default_hugepagesz=1G hugepagesz=1G hugepages=4
#然后执行 重新生成 grub.cfg
#sudo grub-mkconfig -o /boot/grub/grub.cfg

vmware虚拟机环境
vmware虚拟机环境在这是大页时，需要额外增加一个参数"nopku"。
linux16 /vmlinuz-3.10.0-957.el7.x86_64 root=/dev/mapper/centos-root ro crashkernel=auto rd.lvm.lv=centos/root rd.lvm.lv=centos/swap rhgb quiet LANG=en_US.UTF-8 nopku transparent_hugepage=never default_hugepagesz=1G hugepagesz=1G hugepages=2
```

| 参数               | 解释                                                                                                                                                       |
| ------------------ | ---------------------------------------------------------------------------------------------------------------------------------------------------------- |
| default_hugepagesz | 在内核中定义了开机启动时分配的大页面的默认大小。                                                                                                           |
| hugepagesz         | 在内核中定义了开机启动时分配的大页面的大小。<br>可选值为 2MB 和 1GB 。默认是 2MB                                                                           |
| hugepages          | 在内核中定义了开机启动时就分配的永久大页面的数量。默认为 0，即不分配。<br>只有当系统有足够的连续可用页时，分配才会成功。由该参数保留的页不能用于其他用途。 |

## 5.4 查看配置的大页

```
root@server:~# cat /proc/meminfo | grep Huge
AnonHugePages:         0 kB
ShmemHugePages:        0 kB
HugePages_Total:       4
HugePages_Free:        1
HugePages_Rsvd:        0
HugePages_Surp:        0
Hugepagesize:    1048576 kB


如果你的系统正确配置了大页面支持
你应该能够执行以下命令并看到非零值，对应于你的大页面配置
root@server:~/dperf# grep Huge  /proc/meminfo
AnonHugePages:         0 kB
ShmemHugePages:        0 kB
HugePages_Total:       4
HugePages_Free:        1
HugePages_Rsvd:        0
HugePages_Surp:        0
Hugepagesize:    1048576 kB

```

## 5.5 Hugepage 名次解释

| 名次                                   | 解释                                                                                                                                                                                                                                                                                  |
| -------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| HugePages_Total                        | 是大页面池的大小                                                                                                                                                                                                                                                                      |
| HugePages_Free                         | 是池中尚未分配的大页面数。                                                                                                                                                                                                                                                            |
| HugePages_Rsvd                         | 是“reserved”的缩写，是已承诺从池中分配但尚未分配的大页面的数量<br>预留大页面保证应用程序能够在故障时从大页面池中分配一个大页面。                                                                                                                                                      |
| HugePages_Surp                         | 是“surplus”的缩写，是池中大于/proc/sys/vm/nr_hugepages. 剩余巨页的最大数量由 控制 /proc/sys/vm/nr_overcommit_hugepages。注意：当启用释放与每个 hugetlb page 关联的未使用的 vmemmap pages 特性时，当系统处于内存压力时，剩余 huge pages 的数量可能会暂时大于最大剩余 huge pages 数量。 |
| Hugepagesize                           | is the default hugepage size (in kB).                                                                                                                                                                                                                                                 |
| Hugetlb                                | 是所有大小的大页面消耗的内存总量（以 kB 为单位）。如果使用不同大小的大页，这个数字会超过 HugePages_Total \* Hugepagesize。要获得更详细的信息，请参阅 /sys/kernel/mm/hugepages（如下所述）。                                                                                           |
| cat /proc/filesystems &#124; grep huge | 显示内核中配置的“hugetlbfs”类型的文件系统。                                                                                                                                                                                                                                           |
| cat /proc/sys/vm/nr_hugepages          | 指示内核大页面池中“持久”大页面的当前数量。当任务释放时，“持久”大页面将返回到大页面池。具有 root 权限的用户可以通过增加或减少 的值来动态分配更多或释放一些持久性大页面                                                                                                                 |

# 6. 升级系统 python

##6.1 设置软件更新选项
软件和更新下，勾选重要的安全更新和推荐更新

##6.2 安装 python3.9

```
安装python3.9
在系统上安装一些必需的软件包
apt install build-essential software-properties-common libnuma-dev

为系统配置Deadsnakes PPA
sudo add-apt-repository ppa:deadsnakes/ppa  回车确认
sudo apt-get install python3.9
sudo apt-get install -y python3.9-distutils
安装python组件
sudo apt-get install python3.9-dev python3.9-venv python3.9-gdbm python3-pip python3-setuptools

设置默认使用python3.9
$which python3.9
/usr/bin/python3.9
$sudo update-alternatives --install /usr/bin/python3 python3 /usr/bin/python3.6 1
$sudo update-alternatives --install /usr/bin/python3 python3 /usr/bin/python3.9 2
$sudo update-alternatives --config python3

升级pip/setuptools
pip3 install launchpadlib pyelftools
pip3 install --upgrade pip distlib setuptools

## 强制重新安装pip
##python3 -m pip install --upgrade --force-reinstall pip
#root@dperf:~# cd /usr/lib/python3/dist-packages/
#root@dperf:/usr/lib/python3/dist-packages# sudo ln -s apt_pkg.cpython-36m-x86_64-linux-gnu.so apt_pkg.so
#root@dperf:/usr/lib/python3/dist-packages#

```

```
WARNING: Running pip as the 'root' user can result in broken permissions and conflicting behaviour with the system package manager. It is recommended to use a virtual environment instead: https://pip.pypa.io/warnings/venv
python的这个WARNING:可以照此方法解决
python3 -m venv tutorial-env
1.正常
不显示回显
2.不正常
 出错原因：无法创建虚拟环境，因为ensurpip不可用，需要安装python3-venv包
apt install python3.10-venv
source tutorial-env/bin/activate
python -m pip install novas
pip install --upgrade pip

从pip22.1开始，您现在可以使用参数选择退出警告：
pip install --root-user-action=ignore
```

# 7. DPDK 手工下载与安装编译

[下载地址](https://core.dpdk.org/download/)
[官方手册](https://doc.dpdk.org/guides-22.11/)
[先决条件](https://doc.dpdk.org/guides/linux_gsg/sys_reqs.html#bios-setting-prerequisite-on-x86)

digger 注：编译 dperf 之前需要先编译安装 DPDK，
本章节部分推荐使用[Ubuntu install script](#id_666)

```
dpdk 安装先决条件
内核版本 >= 4.4
glibc >= 2.7

root@dperf:~# ldd --version
ldd (Ubuntu GLIBC 2.27-3ubuntu1) 2.27
Copyright (C) 2018 自由软件基金会。
这是一个自由软件；请见源代码的授权条款。本软件不含任何没有担保；甚至不保证适销性
或者适合某些特殊目的。
由 Roland McGrath 和 Ulrich Drepper 编写。
root@dperf:~# uname -a
Linux dperf 4.15.0-20-generic #21-Ubuntu SMP Tue Apr 24 06:16:15 UTC 2018 x86_64 x86_64 x86_64 GNU/Linux
root@dperf:~#
```

##7.1 下载 dpdk

```
root@dperf:~# wget https://fast.dpdk.org/rel/dpdk-22.11.1.tar.xz
root@dperf:~# apt install build-essential git pkg-config  libelf-dev
root@dperf:~# tar -xvf dpdk-22.11.1.tar.xz
root@dperf:~# cd dpdk-stable-22.11.1/

## dpdk 22.03 需要pyelftools
## pip3 install pyelftools --upgrade
##安装meson ninja
root@dperf:~# pip3 install meson ninja
#pip3会将meson软件安装到/home/user/.local/bin
#而通过apt install 安装是使用/usr/bin/meson 且版本比较低，不推荐使用apt install 安装
#所以需要通过修改path路径使得pip安装的meson优先于系统meson被搜索到
#export PATH=~/.local/bin:$PATH
```

## 7.2 编译

```
// 使用选项 -Dexamples 指定编译所有样例程序，在dpdk源码根目录执行
meson -Dbuildtype=debug -Dexamples=ALL -Denable_kmods=true ./aa ##必须指定目标路径
meson build --prefix=/root/dpdk-stable-22.11.1/mydpdk -Denable_kmods=true -Ddisable_libs=""
meson build --prefix=/root/dpdk-stable-22.11.1/mydpdk -Dbuildtype=debug -Dexamples=ALL -Denable_kmods=true -Ddisable_libs=""
log

Build targets in project: 931

DPDK 22.11.1

  User defined options
    prefix      : /root/dpdk-stable-22.11.1/mydpdk
    disable_libs:
    enable_kmods: true


在构建之后的目标文件中ninja install
root@dperf:~/dpdk-stable-22.11.1# cd aa
root@dperf:~/dpdk-stable-22.11.1/aa# ninja -C build install

/bin/sh /root/dpdk-stable-22.11.1/config/../buildtools/symlink-drivers-solibs.sh lib/x86_64-linux-gnu dpdk/pmds-21.0
ldconfig
```

## 7.3 验证 dpdk 安装

```
root@dperf:~# /root/dpdk-stable-22.11.1/build/examples/dpdk-helloworld -l 0-1 -n 2
EAL: Detected 2 lcore(s)
EAL: Detected 1 NUMA nodes
EAL: Detected static linkage of DPDK
EAL: Multi-process socket /var/run/dpdk/rte/mp_socket
EAL: Selected IOVA mode 'VA'
EAL: No available hugepages reported in hugepages-1048576kB
EAL: Probing VFIO support...
EAL: No legacy callbacks, legacy socket not created
hello from core 1
hello from core 0
root@dperf:~#
```

<span id='id_666'/>

# ubuntu install script

使用 dperf 项目的 scripts/build_dpdk_2x.sh 脚本可以一键安装 DPDK-22.11 以及 DPDK-20.11、DPDK-21.11，
下面的脚本在 ubuntu 18.04 上验证通过
脚本包括 dpdk 下载编译及 dperf 下载编译等

```bash
#!/bin/bash

DPDK_VERSION=22.11.1
if [ $# -eq 1 ]; then
    DPDK_VERSION=$1
fi

HOME_DIR=/root
DPDK_XZ=dpdk-${DPDK_VERSION}.tar.xz
DPDK_TAR=dpdk-${DPDK_VERSION}.tar
DPDK_DIR=dpdk-stable-${DPDK_VERSION}

function install_re2c()
{
    cd $HOME_DIR
    wget https://github.com/skvadrik/re2c/releases/download/3.0/re2c-3.0.tar.xz
    tar -xvf re2c-3.0.tar.xz
    cd re2c-3.0
    ./configure
    make
    make install
}

function install_meson
{
    pip3 install meson ninja
}

function install_kernel_dev()
{
    VERSION=`uname -r`
    apt install kernel-devel-$VERSION -y
}

function install_dpdk()
{
    cd $HOME_DIR
    #######################################
    ##if the current directory has a dpdk install package, it will not be download.
     if [ ! -f "$DPDK_XZ" ]; then
     wget http://fast.dpdk.org/rel/$DPDK_XZ
     fi
    #######################################
    tar -xvf $DPDK_XZ
    cd $DPDK_DIR

    # -Dbuildtype=debug
    OPT_LIBS=""
    expr match "${DPDK_VERSION}" "22.11." >/dev/null
    if [ $? -eq 0 ]; then
        OPT_LIBS="-Ddisable_libs=\"\""
    fi
    meson build --prefix=$HOME_DIR/$DPDK_DIR/mydpdk -Dbuildtype=debug -Dexamples=ALL -Denable_kmods=true $OPT_LIBS

    ninja -C build install
    /bin/sh /root/dpdk-stable-22.11.1/config/../buildtools/symlink-drivers-solibs.sh lib/x86_64-linux-gnu dpdk/pmds-21.0
    ldconfig
}

function install_igb_uio()
{
    cd $HOME_DIR
    git clone http://dpdk.org/git/dpdk-kmods
    cd dpdk-kmods/linux/igb_uio/
    make
}

function install_dperf()
{
    cd $HOME_DIR
    git clone https://github.com/baidu/dperf.git
    cd dperf
    export PKG_CONFIG_PATH=/root/dpdk-stable-22.11.1/mydpdk/lib/x86_64-linux-gnu/pkgconfig/
    make
    echo "/root/dpdk-stable-22.11.1/mydpdk/lib/x86_64-linux-gnu/" >> /etc/ld.so.conf
    ldconfig
    #export LD_LIBRARY_PATH=/root/dpdk-stable-22.11.1/mydpdk/lib/x86_64-linux-gnu/
}


pip3 install launchpadlib pyelftools
pip3 install --upgrade pip distlib setuptools
pip3 install meson ninja
apt install build-essential git pkg-config  libelf-dev -y

#install_kernel_dev
install_meson
#install_re2c
install_dpdk
install_igb_uio
install_dperf
```

# 8. dperf 下载与安装编译

2022 年 12 月 14 日，dperf 发布了新版本 v1.4.0 新增支持 DPDK-22.11
本章节部分推荐使用[Ubuntu install script](#id_666)

```
#apt install git pkg-config  libelf-dev
git clone https://github.com/baidu/dperf.git

cd dperf
make
export PKG_CONFIG_PATH=/root/dpdk-stable-22.11.1/mydpdk/lib/x86_64-linux-gnu/pkgconfig/
export LD_LIBRARY_PATH=/root/dpdk-stable-22.11.1/mydpdk/lib/x86_64-linux-gnu/

#export PKG_CONFIG_PATH=/root/dpdk/dpdk-stable-22.11.1/aa/lib/x86_64-linux-gnu/pkgconfig
#export LD_LIBRARY_PATH=/root/dpdk/dpdk-stable-20.11.2/mydpdk/lib64/
#RTE_SDK=/root/dpdk-stable-22.11.1 RTE_TARGET=$TARGET
#TARGET=x86_64-native-linuxapp-gcc
#make -j8
./build/dperf -h #查看帮助信息
root@dperf:~/dperf# ./build/dperf -h                                            -h --help
-v --version
-t --test       Test configure file and exit
-c --conf file  Run with conf file
-m --manual     Show manual
root@dperf:~/dperf# ./build/dperf -v
1.4.0
root@dperf:~/dperf#
#ln -s /usr/bin/python3 /usr/bin/python
```

```
root@dperf:~/dperf# history
pip3 install launchpadlib pyelftools
pip3 install --upgrade pip distlib setuptools
pip3 install meson ninja
apt install build-essential git pkg-config  libelf-dev
tar -xvf dpdk-22.11.1.tar.xz
cd dpdk-stable-22.11.1/
meson build --prefix=/root/dpdk-stable-22.11.1/mydpdk -Denable_kmods=true -Ddisable_libs=""
ninja -C build install
cd mydpdk/lib/x86_64-linux-gnu/pkgconfig/
cd
git clone https://github.com/baidu/dperf.git
cd dperf
##LD_LIBRARY_PATH是临时的，写入ld.so.conf是永久的
##echo "/root/dpdk-stable-22.11.1/mydpdk/lib/x86_64-linux-gnu/" >> /etc/ld.so.conf
##ldconfig 后生效
export PKG_CONFIG_PATH=/root/dpdk-stable-22.11.1/mydpdk/lib/x86_64-linux-gnu/pkgconfig/
make
export LD_LIBRARY_PATH=/root/dpdk-stable-22.11.1/mydpdk/lib/x86_64-linux-gnu/
./build/dperf -h ##查看dperf帮助信息
./build/dperf -v ##查看dperf版本信息
```

## 8.1 如何验证 dperf 成功启动

配置 dperf 用 server 模式运行，此时 dperf 不会主动发报文，我们可以

1. ping dperf 的接口地址（'port'配置项的地址），判断网络的连通性
2. ping dperf 的监听地址（'server'配置项的地址），判断网络的连通性
3. curl dperf 的监听地址+监听端口
   注意：dperf 服务器只会响应'client'中的地址发出的 TCP/UDP 请求，在使用 curl 做测试时，需要把客户端的 IP 地址加入配置文件，dperf 作为服务器运行时，可以配置多条'client'

# 9. DPDK 绑定网卡

https://git.dpdk.org/dpdk-kmods/tree/windows/netuio/README.rst

##9.1 为什么要绑定网卡？
DPDK 让用户态程序可以直接控制网卡收发报文，以达到极高的网络处理性能。这意味着，我们要把网卡从操作系统摘除，绑定用户态驱动。结果是，这些网卡从操作系统'消失'，ssh 不能使用这个网卡。
管理面网卡与数据面网卡
运行 DPDK 程序的网卡称为数据面网卡，用于 ssh 登录等管理用途的网卡称为管理面网卡。使用 dperf 的服务器上必须要预留一张网卡为管理面网卡。

https://doc.dpdk.org/guides/linux_gsg/linux_drivers.html

## 9.2 查看 dpdk 是否支持该网卡

```
#查询网卡的devid号
lspci -nn | grep Ethernet
02:01.0 Ethernet controller [0200]: Intel Corporation 82545EM Gigabit Ethernet Controller (Copper) [8086:100f] (rev 01)
02:06.0 Ethernet controller [0200]: Intel Corporation 82545EM Gigabit Ethernet Controller (Copper) [8086:100f] (rev 01)

#在dpdk代码中搜索此devid号，
grep --include=*.h -rn -e '100f'

dpdk-stable-20.11.1/drivers/net/bnx2x/ecore_reg.h:1274:#define NIG_REG_BRB0_OUT_EN                                         0x100f8
dpdk-stable-20.11.1/drivers/net/bnx2x/ecore_reg.h:2112:#define NIG_REG_XCM0_OUT_EN                                         0x100f0
dpdk-stable-20.11.1/drivers/net/bnx2x/ecore_reg.h:2114:#define NIG_REG_XCM1_OUT_EN                                         0x100f4
dpdk-stable-20.11.1/drivers/net/bnx2x/ecore_hsi.h:2669:        #define SHMEM_AFEX_VERSION_MASK                  0x100f
dpdk-stable-20.11.1/drivers/common/sfc_efx/base/efx_regs_mcdi.h:349:#define        MC_CMD_ERR_NO_MAC_ADDR 0x100f
是Intel支持的一种虚拟网卡，可以绑定的
```

## 9.3 DPDK 支持的网卡列表

https://core.dpdk.org/supported/nics/

## 9.4 查看网卡编号

```
lshw -businfo -c network
/root/dpdk-stable-22.11.1/usertools/dpdk-devbind.py -s
lspci -v -s 02:00.0
```

## 9.5 下载安装网卡 igb_uio 驱动

[Ubuntu install script](#id_666) 中已经包含

## 9.6 用 igb_uio 绑定网卡

```
ifconfig eth0 down
dpdk-devbind.py -b igb_uio 0000:02:05.0
```

## 9.7 查看绑定结果

```
root@dperf:~/dpdk-stable-22.11.1/usertools# ./dpdk-devbind.py -b igb_uio 0000:02:05.0
root@dperf:~/dpdk-stable-22.11.1/usertools# ./dpdk-devbind.py -s
Network devices using DPDK-compatible driver
============================================
0000:02:05.0 '82545EM Gigabit Ethernet Controller (Copper) 100f' drv=igb_uio unused=e1000
Network devices using kernel driver
===================================
0000:02:01.0 '82545EM Gigabit Ethernet Controller (Copper) 100f' if=eth0 drv=e1000 unused=igb_uio *Active*
No 'Baseband' devices detected
==============================
No 'Crypto' devices detected
============================
No 'DMA' devices detected
=========================
No 'Eventdev' devices detected
==============================
No 'Mempool' devices detected
=============================
No 'Compress' devices detected
==============================
No 'Misc (rawdev)' devices detected
===================================
No 'Regex' devices detected
===========================
root@dperf:~/dpdk-stable-22.11.1/usertools#
```

## 9.8 使用参数“wc_activate=1”加载 igb_uio 以提高性能。

insmod /root/dpdk/dpdk-stable-22.11.1/x86_64-native-linuxapp-gcc/kmod/igb_uio.ko wc_activate=1

## 9.9 其他网卡本例不涉及

# 10 重启之后注意事项

纯自用,可以考虑加入 rc.local 自动添加

```
modprobe uio
insmod /root/dpdk-kmods/linux/igb_uio/igb_uio.ko
ifconfig eth3 down
ifconfig eth4 down
./dpdk-stable-22.11.1/usertools/dpdk-devbind.py -b igb_uio 0000:05:00.0
./dpdk-stable-22.11.1/usertools/dpdk-devbind.py -b igb_uio 0000:05:00.1
./dperf/build/dperf -c ./dperf/digger/client.conf
./dperf/build/dperf -c ./dperf/digger/server.conf
```

# 11 KNI 部分【本例不涉及】

参考文档
[kni1](https://doc.dpdk.org/guides-20.11/nics/kni.html)
[kni2](https://doc.dpdk.org/guides/nics/kni.html)

# 12 实战部分--配置相关

## 12.1 名次解释

| 参数 | 解释                                                                                                                                                     |
| ---- | -------------------------------------------------------------------------------------------------------------------------------------------------------- |
| bPS  | bits Per Second;即每秒发送多少比特 =BPS/8                                                                                                                |
| BPS  | Bytes Per Second;即每秒发送多少字节                                                                                                                      |
| PPS  | Packet Per Second(包每秒)，网络设备的转发性能以“包转发性能”来表示，即设备在单位时间内能够处理多少个“包”，即每秒发送多少个分组数据包                      |
| Mbps | Million bits per second = 1,000,000 bps                                                                                                                  |
| RPS  | "received per second 每秒接收数量（RPS）是指每秒钟能够接收的数据包或事务的数量, PPS 和 RPS 的换算是一对一的，一个 RPS 等于一个 PPS，所以 1RPS 等于 1PPS" |
| TPS  | Transactions Per Second;即每秒完成多少次发送过程                                                                                                         |

## 12.2 配置关键参数

https://github.com/baidu/dperf/blob/main/docs/configuration-CN.md

## 12.3 配置参数中的必须项

| parameter | syntax                          | default:                            | required | mode           |
| --------- | ------------------------------- | ----------------------------------- | -------- | -------------- | -------------- |
| cps       | cps Number                      | -                                   | yes      | client         |
| client    | client IPAddrStart IPAddrNumber | -                                   | yes      | client, server |
| cpu       | cpu n0 n1 n2-n3...              | -                                   | yes      | client, server |
| duration  | duration Time                   | 100s                                | yes      | client, server |
| listen    | listen Port Number              | 80 1                                | yes      |                |
| mode      | client                          | server                              | -        | yes            |                |
| port      | port PCI                        | BOND IPAddress Gateway [GatewayMAC] | -        | yes            | client, server |
| server    | server IPAddrStart IPAddrNumber | -                                   | yes      | client, server |

## 12.4 全部参数表

[详细解释见](https://github.com/baidu/dperf/blob/main/docs/statistics-CN.md)

| parameter      | syntax                                                                           | default | required | mode           |
| -------------- | -------------------------------------------------------------------------------- | ------- | -------- | -------------- |
| cc             | cc Number                                                                        | -       | no       | client         |
| change_dip     | change_dip IPAddress Step Number                                                 | -       | no       | client         |
| client         | client IPAddrStart IPAddrNumber                                                  | -       | yes      | client, server |
| client_hop     | client_hop                                                                       | -       | no       | client         |
| cps            | cps Number                                                                       | -       | yes      | client         |
| cpu            | cpu n0 n1 n2-n3...                                                               | -       | yes      | client, server |
| daemo          | daemo                                                                            | -       | no       | client, server |
| duration       | duration Time                                                                    | 100s    | yes      | client, server |
| flood          | flood                                                                            | -       | no       | client         |
| http_host      | http_host String(1-127)                                                          | dperf   | no       | client         |
| http_path      | http_path String(1-255)                                                          | /       | no       | client         |
| lport_range    | lport_range Number [Number]                                                      | -       | no       | client         |
| launch_num     | launch_num Number                                                                | 4       | no       | client         |
| jumbo          | jumbo                                                                            | -       | no       | client, server |
| keepalive      | keepalive interval(timeout) [num]                                                | -       | no       | client, server |
| kni            | kni [ifName]                                                                     | -       | no       | client, server |
| launch_num     | launch_num Number                                                                | 4       | no       | client         |
| listen         | listen Port Number                                                               | 80 1    | yes      |                |
| lport_range    | lport_range Number [Number]                                                      | 1 65535 | no       | client         |
| mode           | client/server                                                                    | -       | yes      | -              |
| mss            | mss Number                                                                       | 1460    | no       | client, server |
| payload_size   | payload_size Number(>=1)                                                         | -       | no       | client, server |
| payload_random | payload_random                                                                   | -       | no       | client, server |
| packet_size    | pakcet_size Number(0-1514)                                                       | -       | no       | client, server |
| port           | port PCI BOND IPAddress Gateway [GatewayMAC]                                     | -       | yes      | client, server |
| protocol       | protocol tcp udp http                                                            | tcp     | no       | client, server |
| quiet          | quiet                                                                            | -       | no       | client, server |
| rss            | rss [l3/l3l4/auto] [mq_rx_none mq_rx_rss]                                        | -       | no       | client, server |
| server         | server IPAddrStart IPAddrNumber                                                  | -       | yes      | client, server |
| slow_start     | slow_start Seconds(10-600)                                                       | 30      | no       | client         |
| socket_mem     | socket_mem n0,n1,n2...                                                           | -       | no       | client, server |
| tcp_rst        | tcp_rst Number[0-1]                                                              | 1       | no       | client, server |
| tx_burst       | tx_burst Number(1-1024)                                                          | 8       | no       | client, server |
| tos            | tos Number(0x00-0xff or 0-255)                                                   | 0       | no       | client, server |
| vxlan          | vxlan vni innerSMAC innerDMAC localVtep<br>IPAddr Number remoteVtepIPAddr Number | -       | no       | client, server |
| wait           | wait Seconds                                                                     | 3       | no       | client         |

## 12.5 给网关配置 mac 地址

```
#port         pci             addr      gateway    [mac]
port       0000:01:00.0    6.6.245.3 6.6.245.1  b4:a9:fc:ab:7a:85
```

## 12.6 FDIR

网卡支持 FDIR
通常物理网卡支持 FDIR，使用 FDIR 特性，可以发挥 dperf 的最佳性能。CPU 数量需要与 server 的 IP 数量一致。

```
cpu          0 1 2 3 4 5 6 7
server       192.168.31.50   8
```

## 12.7 L3L4RSS

通常虚拟环境里不能设置 FDIR，但是可以使用 RSS，根据网卡特性使用 L3 或 L3L4 哈希策略，要求 server IP 的数量与 CPU 个数一致，client 需要配置足够的 IP，其 IP 在会在 CPU 间平分

```
cpu    0 1 2 3
rss    auto
client 192.168.1.100 10
server 192.168.1.200 4
```

# 13 实战部分-测试

下列举例中采用一台物理机,绑定两块网卡的配置,测试的时候最好可以同时启动 client 和 server
由于 client 测有默认的 slow_start:30 seconds 的等待时间
因此 server 上的 duration 时间建议比 client 侧的长 30 秒

## 13.1 Throughput

1. 增大报文长度，payload_size 最小为 1,最大支持设置为 1400,超过会报错
2. 降低并发连接数，如 cc 设置为 3000
3. 提升请求发送速度，如从 60 秒发送 1 次，提升为 1 毫秒发送一次
4. 降低 cps，由于并发度连接数较低，应该把新建速度同时调低

```
mode         client
socket_mem   1024
cpu          0
duration     60s
cps          500
payload_size 1
keepalive    1ms
cc           10000
port        0000:05:00.1    192.168.10.20 192.168.10.101
client      192.168.10.20      20
server      192.168.10.101     2
listen      80                 1
————————————————————————————————————————————————————————
mode         server
socket_mem   1024
cpu          1
duration     90s
payload_size 1400
keepalive    1s
port        0000:05:00.0    192.168.10.101 192.168.10.20
client      192.168.10.20       20
server      192.168.10.101      2
listen      80                  1
```

结果查看
<mark>注意: 当前版本的 dperf 测试结果没有写入数据库,不像使用商用测试仪测完会有丰富的报表可供查看
且 Test Finished 部分为测试过程数据值的求和, 因此查看测试的吞吐需要查看测试过程中的打印信息
</mark>
![client-throughput](/dperf-img/client-throughput.png)

server

![server-throughput](/dperf-img/server-throughput.png)

|        | bitsRx      | bitsTx        | bitsRx+bitsTx bps | Mbps        |
| ------ | ----------- | ------------- | ----------------- | ----------- |
| client | 957,315,728 | 86,612,016    | 1043927744        | 1043.927744 |
| server | 86,636,816  | 1,015,115,136 | 1101751952        | 1101.751952 |

dperf 每秒输出的 bitsRx 与 bitsTx，这就是每秒上行带宽与下行带宽，bitsRx+bitsTx 就是总带宽,<br>单位是比特每秒(bits/s)。注意，测试结束后的 bitRx 与 bitsTx 是 total 值不是速度值。

补充两个举例,供参考<br>
例 1：测试 8Gbps 双向带宽应该如何配置客户端
计算方法 8Gbps ~= 1 个 CPU x 1000 字节 x 8bits x 1000 请求/秒 \* 1000 个连接

```
mode client
tx_burst 4
cc 1000
cps 1000
pakcet_size 1000
keepalive 1ms
cpu 0
```

例 2：测试 100Gbps 双向带宽应该如何配置客户端
计算方法 100Gbps ~= 4 个 CPU x 1000 字节 x 8bits x 1000 请求/秒 \* 3200 个连接

```
mode client
tx_burst 16
cc 3200
cps 1000
pakcet_size 1000
keepalive 1ms
cpu 0 1 2 3
```

## 13.2 CPS

1. 新建连接数测试不能开启 keepalive
2. 设置 HTTP 响应长度为最小：payload_size 1
   在新建连接数测试中，我们使用短连接，每个连接只一个 HTTP 请求与响应，报文序列如下：

```
客户端发送SYN
服务器发送SYN+ACK
客户端发送请求（最小HTTP请求）
客户端发送请求（最小HTTP响应）
服务器发送响应+FIN
客户端发送FIN + ACK
服务器发送ACK
```

新建连接数测试会消耗较多的连接数（五元组），在新建连接数测试中，建议 dperf 连接总数是新建连接数目标的 10 倍以上；因为 tcp 超时为 2 秒，4 次超时则认为连接失败。

dperf 中连接总数 = 客户端 IP 数 _ 65535 _ 服务器 IP 数 \* 监听端口数

```
mode          client
socket_mem    1024
cpu           0
duration      60s
cps           500k
payload_size  1
#keepalive    1ms
#protocol     udp
#cc           10000
port        0000:05:00.1    192.168.10.20 192.168.10.101
client      192.168.10.20      20
server      192.168.10.101     1
listen      80                 2
————————————————————————————————————————————————————————
mode          server
socket_mem    1024
cpu           1
duration      90s
payload_size  1
#keepalive    1s
#protocol     udp
port        0000:05:00.0    192.168.10.101 192.168.10.20
client      192.168.10.20       20
server      192.168.10.101      1
listen      80                  2
```

测试结果
可以看到 CPS 结果为在 400k 左右
client
![client-cps](/dperf-img/client-cps.png)

server

![server-cps](/dperf-img/server-cps.png)

## 13.3 CC

在并发连接数测试中，我们要使用长连接，每个连接持续发送 HTTP 请求、响应。在测试过程中，并发连接数持续增加，直到达到测试目标；我们可以通过客户端的每秒新建连接数来控制爬坡斜率；在测试并发连接数时，建议为客户端配置较大请求发送的间隔；payload_size 设置为最小。

```
mode          client
socket_mem    1024
cpu           0
duration      60s
cps           500k
payload_size  1
keepalive     1ms
#protocol     udp
cc            200k
port        0000:05:00.1    192.168.10.20 192.168.10.101
client      192.168.10.20     20
server      192.168.10.101     1
listen      80              2
————————————————————————————————————————————————————————
mode          server
socket_mem    1024
cpu           1
duration      90s
payload_size  1
#keepalive    1s
#protocol     udp
port        0000:05:00.0    192.168.10.101 192.168.10.20
client      192.168.10.20       20
server      192.168.10.101      1
listen      80                  2
```

测试结果
可以看到 client 侧已经发出 200k, server 侧峰值在 139k
![client-cc](/dperf-img/client-cc.png)

server 侧

![server-cc](/dperf-img/server-cc.png)

## 13.4 udp pps

PPS（packets perf second）是网络设备的一个重要指标，
测试 PPS 只需要在并发连接数的基础上做少量调整，与带宽测试类似。

1. 降低报文长度，payload_size 设置为 1
2. 设置合适的并发连接数
3. 提升请求发送速度，如从 60 秒发送 1 次，提升为 1 毫秒发送一次
4. 降低 cps，由于并发度连接数较低，应该把新建速度同时调低

```
mode          client
socket_mem    1024
cpu           0
duration      30s
cps           100
payload_size  1400
keepalive     1ms
protocol      udp
cc            100
port        0000:05:00.1    192.168.10.20 192.168.10.101
client      192.168.10.20      2
server      192.168.10.101     1
listen      80                 2
-----------------------------------------------------------
mode           server
socket_mem     1024
cpu            1
duration       60s
payload_size   1400
keepalive      1s
protocol       udp
port        0000:05:00.0    192.168.10.101 192.168.10.20
client      192.168.10.20       2
server      192.168.10.101      1
listen      80                  2
```

测试结果

![client-pps](/dperf-img/client-udppps.png)

server 侧

![server-pps](/dperf-img/server-udppps.png)

## 13.5 ipv6 cps

```
mode          client
socket_mem    1024
cpu           0
duration      60s
cps           500k
payload_size  1
#keepalive    1ms
#protocol     udp
#cc          10000
port        0000:05:00.1    2001:1::10 2001:1::100
client      2001:1::10      20
server      2001:1::100     1
listen      80              2
----------------------------------------------------
mode          server
socket_mem    1024
cpu           1
duration      90s
payload_size  1
#keepalive    1s
#protocol       udp
port        0000:05:00.0    2001:1::100 2001:1::10
client      2001:1::10       20
server      2001:1::100      1
listen      80               2
```

测试结果

![client-ipv6](/dperf-img/client-ipv6cps.png)

server 侧

![server-ipv6](/dperf-img/server-ipv6cps.png)

## 13.7 其他宣称支持，但本文暂不涉及的项(欢迎补充)

详见
https://github.com/baidu/dperf/tree/main/test

RTT(Round-Trip Time)
bond
kni
kni-ipv6
vxlan
vxlan-ipv6
[大象流](https://zhuanlan.zhihu.com/p/609817366)

# 14. dperf 统计信息说明

https://github.com/baidu/dperf/blob/main/docs/statistics-CN.md

## 关键统计信息

| 参数   | 说明                                                                 |
| ------ | -------------------------------------------------------------------- |
| bitsRx | 每秒接收到的 bit。 （发送+接受的=总带宽 throughput）                 |
| bitsTx | 每秒发送的 bit。                                                     |
| skOpen | 每秒打开的 socket 个数, 就是每秒新建连接数(CPS)。                    |
| skCon  | 当前处于打开状态的 socket 个数，即并发连接数。Concurrent connections |
| udpRx  | 每秒收到的 UDP 报文个数                                              |
| udpTx  | 每秒发送的 UDP 报文个数                                              |

## 全部统计信息

| 参数     | 说明                                                                                                                                                                  |
| -------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| ackRt    | 每秒重传 TCP ACK 报文的个数，重传意味着丢包。                                                                                                                         |
| arpRx    | 每秒收到的 arp 报文数。                                                                                                                                               |
| arpTx    | 每秒发送的 arp 报文数。                                                                                                                                               |
| badRx    | 每秒收到的错误报文数，错误原因：checksum 错误 ipv4 头部长度不为 20                                                                                                    |
| bitsRx   | 每秒接收到的 bit。 （发送+接受的=总带宽 throughput）                                                                                                                  |
| bitsTx   | 每秒发送的 bit。                                                                                                                                                      |
| cpuUsage | 每个 dperf worker 线程的 CPU 使用率。                                                                                                                                 |
| dropTx   | 由于一次发送大量报文导致部分报文未发送引起的每秒丢表数。                                                                                                              |
| finRt    | 每秒重传 TCP FIN 报文的个数，重传意味着丢包。                                                                                                                         |
| finRx    | 每秒接收到的 TCP FIN 报文个数。                                                                                                                                       |
| finTx    | 每秒发送的 TCP FIN 报文个数。                                                                                                                                         |
| http2XX  | 客户端每秒接收的 HTTP 2XX 响应个数,服务端每秒发送的 HTTP 2XX 响应个数                                                                                                 |
| httpErr  | 每秒收到的 HTTP 报文错误个数，错误原因<br>客户端收到的响应报文不是 HTTP 2XX 服务器收到的请求不是 HTTP GET                                                             |
| httpGet  | "客户端每秒发送的 HTTP GET 请求个数 服务端每秒收到的 HTTP GET 请求个数"                                                                                               |
| icmpRx   | 每秒收到的 icmp 报文数量，包含 icmpv6 报文。                                                                                                                          |
| icmpTx   | 每秒发送的 icmp 报文数量，包含 icmpv6 报文。                                                                                                                          |
| ierrors  | 网卡接收错误的累计数。                                                                                                                                                |
| imissed  | 由于接收 buffer 不够，导致网卡硬件丢包的累计数。                                                                                                                      |
| oerrors  | 网卡发送错误的累计数。                                                                                                                                                |
| otherRx  | 每秒收到的未知类型的报文个数。                                                                                                                                        |
| pktRx    | 每秒接收的报文个数。                                                                                                                                                  |
| pktTx    | 每秒发送的报文个数。                                                                                                                                                  |
| pushRt   | 每秒重传 TCP PUSH 报文的个数，重传意味着丢包。                                                                                                                        |
| rstRx    | 每秒接收到的 TCP RST 报文个数。                                                                                                                                       |
| rstTx    | 每秒发送的 TCP RST 报文个数。                                                                                                                                         |
| rtt(us)  | 首包平均 rtt，单位是 us。                                                                                                                                             |
| seconds  | 程序启动到现在的秒数。                                                                                                                                                |
| skClose  | 每秒关闭的 socket 个数。                                                                                                                                              |
| skCon    | 当前处于打开状态的 socket 个数，即并发连接数。Concurrent connections                                                                                                  |
| skErr    | 每秒出错的并发连接数，出错原因：重传超过 3 次                                                                                                                         |
| skOpen   | 每秒打开的 socket 个数, 就是每秒新建连接数(CPS)。                                                                                                                     |
| synRt    | 每秒重传 TCP SYN 报文的个数，重传意味着丢包。                                                                                                                         |
| synRx    | 每秒接收到的 TCP SYN 报文个数。                                                                                                                                       |
| synTx    | 每秒发送的 TCP SYN 报文个数。                                                                                                                                         |
| tcpDrop  | 每秒丢掉的 TCP 报文的个数，原因包括:<br>错误的目的端口<br>找不到连接<br>错误的 TCP 状态<br>错误的 TCP 序列号<br>错误的 TCP Flag<br>服务器运行 UDP 协议，收到 TCP 报文 |
| tcpRx    | 每秒收到的 TCP 报文个数。                                                                                                                                             |
| tcpTx    | 每秒收到的 TCP 报文个数。                                                                                                                                             |
| tosRx    | 每秒收到的 IP 报文中 tos 与配置中 tos 相等的报文个数。                                                                                                                |
| udpDrop  | 每秒 UDP 报文的丢包数。。                                                                                                                                             |
| udpRt    | 每秒 UDP 报文的重传数。                                                                                                                                               |
| udpRx    | 每秒收到的 UDP 报文个数。                                                                                                                                             |
| udpTx    | 每秒发送的 UDP 报文个数。                                                                                                                                             |

# 15 dperf 启动报错信息排查

```
EAL: Detected CPU lcores: 2
EAL: Detected NUMA nodes: 1
EAL: Detected shared linkage of DPDK
EAL: Multi-process socket /var/run/dpdk/rte/mp_socket
EAL: Selected IOVA mode 'PA'
EAL: No free 2048 kB hugepages reported on node 0
EAL: FATAL: Cannot get hugepage information.
EAL: Cannot get hugepage information.
rte_eal_init fail
dpdk_eal_init fail
dpdk init fail
root@dperf:~/dperf#

没有配置hugepage导致
```

```
line6:error

配置错误，请查看是否缺少12.2下参数中的必须项
```

```
root@dperf:~/dperf# ./build/dperf -c ./digger/server.conf
EAL: Detected CPU lcores: 2
EAL: Detected NUMA nodes: 1
EAL: Detected shared linkage of DPDK
EAL: Cannot create lock on '/var/run/dpdk/rte/config'. Is another primary process running?
EAL: FATAL: Cannot init config
EAL: Cannot init config
rte_eal_init fail
dpdk_eal_init fail
dpdk init fail
root@dperf:~/dperf#

ctrl+c 后再次启动容易出现下面的错误
已经存在其他进程
kill后在启动即可
```

```
Error: insufficient sockets. worker=0 sockets=65535 cps's cc=2000000

套接字不足,一般来说是cps或cc参数配置的过大,而ip数不足导致
```

```
Error: insufficient sockets. worker=0 sockets=65535 cc=100000

地址配少了，达不到这个目标，要多配一些client ip，我一般client 配100个ip
```

```
root@server:~# ./dperf/build/dperf -c ./dperf/digger/client-throughput.conf
EAL: Detected CPU lcores: 8
EAL: Detected NUMA nodes: 1
EAL: Detected shared linkage of DPDK
EAL: Multi-process socket /var/run/dpdk/2503/mp_socket
EAL: Selected IOVA mode 'PA'
EAL: Probe PCI driver: net_ixgbe (8086:10fb) device: 0000:05:00.1 (socket -1)
TELEMETRY: No legacy callbacks, legacy socket not created
socket allocation failed, size 0.15GB num 1966050
work space init error

error: Segmentation fault

驱动不对，换成igb_uio驱动
```

```
bad queue_num 2 max rx 1 max tx 1
Error: 'rss' is required if cpu num is not equal to server ip num

这俩我也不知道是啥原因,但肯定是配置参数不正确导致的
```

## 其他可能遇到的错误,欢迎补充

```
keepalive request interval must be a multiple of 10us
microseconds can only be used if the interval is less than 1 millisecond
duplicate vlan
duplicate rss
unknown rss type \'%s\'\n", argv[1]);
rss type \'%s\' dose not support \'mq_rx_none\'\n", argv[1]);
unknown rss config \'%s\'\n", argv[2]);
duplicate quiet
'vxlan' requires cpu num to be equal to server ip num
cpu num less than server ip num at 'client' mode;
'rss' is required if cpu num is not equal to server ip num
Cannot enable vlan and vxlan at the same time
Cannot enable vlan and bond at the same time
client and server address conflict
local ip conflict with client address
gateway ip conflict with server address
local ip conflict with server address
gateway ip conflict with client address
wait in server config
slow_start in server config\n");
no targets
insufficient sockets. worker=%d sockets=%u cc=%lu\n", i, socket_num, cc);
insufficient sockets. worker=%d sockets=%u cps's cc=%lu\n", i, socket_num, cps_cc);
both payload_size and packet_size are set
big packet_size %d\n", cfg->packet_size);
big payload_size %d\n", cfg->payload_size);
bad mss %d\n", cfg->mss)
rss is not supported for vxlan.
'rss auto|l3\' conflicts with \'flood\'
rss \'auto\' requires one server address.
'cc' requires 'keepalive'
'change_dip\' only support client mode
'change_dip\' only support flood mode
'change_dip\' not support vxlan
bad ip address family of \'change_dip\'
number of \'change_dip\' is less than cpu number
The HTTP host/path cannot be set with packet_size or payload_size.
the HTTP host/path cannot be set in server mode.
The HTTP host/path cannot be set in udp protocol.
```
