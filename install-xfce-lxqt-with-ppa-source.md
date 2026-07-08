#  Install xfce 4.18 with ppa source

```
add-apt-repository ppa:xubuntu-dev/staging  开发版

#add-apt-repository ppa:xubuntu-dev/sru-staging 稳定版
#add-apt-repository ppa:xubuntu-dev/ppa  每日构建
#add-apt-repository ppa:xubuntu-dev/extras 预览版 软件包尚未经过主要测试
#add-apt-repository ppa:xubuntu-dev/experimental 实验性质 通常是下个测试版本

apt update

apt install -y lightdm 
apt install -y xubuntu-core 

#full-install 
#apt install -y xubuntu-desktop

```

## How to Restore

```
apt install ppa-purge && sudo ppa-purge ppa:xubuntu-dev/staging
```

# Install lxqt 1.3.0 with ppa source 


```
#need ubuntu 22.04
add-apt-repository ppa:lubuntu-dev/backports
apt update
apt install -y sddm 
apt install -y lxqt-core
#full-install
#apt install -y lubuntu-desktop

```

# Memory Usage Comparison
```
apt install lightdm xubuntu-core 
root@test:~# apt list --installed | wc -l

WARNING: apt does not have a stable CLI interface. Use with caution in scripts.

1625
root@test:~# free -h
               total        used        free      shared  buff/cache   available
Mem:           3.8Gi       752Mi       156Mi       3.0Mi       2.9Gi       2.8Gi
Swap:          2.9Gi       1.0Mi       2.9Gi
root@test:~#
########################################################################################
apt install lxdm xubuntu-core
root@test:~# apt list --installed | wc -l

WARNING: apt does not have a stable CLI interface. Use with caution in scripts.

1620
root@test:~# free -h
               total        used        free      shared  buff/cache   available
Mem:           3.8Gi       680Mi       2.6Gi       2.0Mi       508Mi       2.9Gi
Swap:          2.9Gi          0B       2.9Gi
root@test:~#
#########################################################################################

sddm xfwm4 lxqt-core 
root@test:~# apt list --installed | wc -l

WARNING: apt does not have a stable CLI interface. Use with caution in scripts.

1010
root@test:~# free -h
               total        used        free      shared  buff/cache   available
Mem:           3.8Gi       506Mi       669Mi       3.0Mi       2.6Gi       3.0Gi
Swap:          2.9Gi          0B       2.9Gi
root@test:~#
#########################################################################################
lxdm xfwm4 lxqt-core
root@test:~# apt list --installed | wc -l

WARNING: apt does not have a stable CLI interface. Use with caution in scripts.

979
root@test:~# free -h
               total        used        free      shared  buff/cache   available
Mem:           3.8Gi       496Mi       2.7Gi       3.0Mi       569Mi       3.1Gi
Swap:          2.9Gi          0B       2.9Gi
root@test:~#
```

# lxdm command

```
cat /etc/lxdm/lxdm.conf
session=/usr/bin/xfwm4
apt-get install xinit
echo "exec startlxqt" >> ~/.xinitrc
apt -y install ttf-wqy-microhei
apt -y install lxqt-admin software-properties-qt software-properties-qt
lubuntu-update-notifier

# edit /etc/lxdm/lxdm.conf and change the name of the theme.
# Themes are stored in /usr/share/lxdm/themes
```