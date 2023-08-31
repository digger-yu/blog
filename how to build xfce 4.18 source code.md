```
#!/bin/bash
echo "deb http://mirrors.tuna.tsinghua.edu.cn/ubuntu lunar main restricted universe" >> /etc/apt/sources.list
echo "PKG_CONFIG_PATH=/usr/local/lib/pkgconfig/" >> /root/.bashrc
echo "export PKG_CONFIG_PATH" >> /root/.bashrc
source .bashrc

apt update && apt dist-upgrade -y --fix-broken
apt --fix-broken install -y 

apt install -y libgtk-3-dev libgtk-4-dev intltool libgudev-1.0-dev libwnck-3-dev libupower-glib-dev libglib2.0-dev libmount-dev libselinux1-dev libnotify-dev build-essential
apt --fix-broken install -y
apt autoremove -y
wget https://archive.xfce.org/xfce/4.18/fat_tarballs/xfce-4.18.tar.bz2
tar -xvf xfce-4.18.tar.bz2
cd src
 for i in `ls *.tar.bz2| awk '{print $NF}'`; do tar xvjf $i; done && rm -rf *.bz2

cd xfce4-dev-tools-4.18.0/
./configure --prefix=/usr/local/ && make && make install 

cd ../libxfce4util-4.18.0/
./configure --prefix=/usr/local/ && make && make install

cd ../xfconf-4.18.0/
./configure --prefix=/usr/local/ && make && make install

cd ../libxfce4ui-4.18.0/
./configure --prefix=/usr/local/ && make && make install

cd ../garcon-4.18.0/
./configure --prefix=/usr/local/ && make && make install

cd ../exo-4.18.0/
./configure --prefix=/usr/local/ && make && make install.

cd ../thunar-4.18.0/
./configure --prefix=/usr/local/ && make && make install

cd ../thunar-volman-4.18.0/
./configure --prefix=/usr/local/ && make && make install

cd ../xfce4-panel-4.18.0/
./configure --prefix=/usr/local/ && make && make install

cd ../xfce4-settings-4.18.0/
./configure --prefix=/usr/local/ && make && make install

cd ../xfce4-power-manager-4.18.0/
./configure --prefix=/usr/local/ && make && make install

cd ../xfce4-session-4.18.0/
./configure --prefix=/usr/local/ && make && make install

cd ../xfdesktop-4.18.0/
./configure --prefix=/usr/local/ && make && make install

cd ../xfwm4-4.18.0/
./configure --prefix=/usr/local/ && make && make install

cd ../xfce4-appfinder-4.18.0/
./configure --prefix=/usr/local/ && make && make install

cd ../tumbler-4.18.0/
./configure --prefix=/usr/local/ && make && make install

echo "########## ALL WORK DONE.########## " 

systemctl start lightdm.service

#plank Only X11 environments are supported.
解决：删除 /etc/gdm3/custom.conf中#WaylandEnable=false 的注释重启即可
##命令运行语言支持检查
#apt install $(check-language-support)
查看您当前是 Xorg(X11) 还是 Wayland
#echo $XDG_SESSION_TYPE
#打开“系统设置”，左边找到“关于本系统”
#查看当前的显示管理器
#cat /etc/X11/default-display-manager
#切换显示管理器
#如:sudo dpkg-reconfigure sddm
```

