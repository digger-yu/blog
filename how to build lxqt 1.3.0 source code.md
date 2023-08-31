https://github.com/lxqt/lxqt/wiki/Building-from-source

```
#!/bin/bash
echo "##########install CMAKE##########"
apt install -y build-essential cmake git

echo "############install QT5############"
apt install -y qtbase5-private-dev libqt5svg5-dev qttools5-dev libqt5x11extras5-dev libpolkit-qt5-1-dev

echo "##########install KDE components##########"
apt-get install -y libkf5guiaddons-dev libkf5idletime-dev libkf5screen-dev libkf5windowsystem-dev libkf5solid-dev

echo "##########install Other ##########"
apt install -y bash libgtk2.0-bin hicolor-icon-theme libasound2-dev libconfig-dev libdbusmenu-qt5-dev libexif-dev libfm-dev libjson-glib-dev libmenu-cache-dev libmuparser-dev libpolkit-agent-1-dev libpolkit-qt5-1-dev libpulse-dev libsensors4-dev libstatgrab-dev libudev-dev libupower-glib-dev libx11-xcb-dev libxcb-composite0-dev libxcb-damage0-dev libxcb-dpms0-dev libxcb-image0-dev libxcb-randr0-dev libxcb-screensaver0-dev libxcb-util0-dev libxcomposite-dev libxcursor-dev libxdamage-dev libxi-dev libxkbcommon-x11-dev libxss-dev libxtst-dev lxmenu-data openbox-dev oxygen-icon-theme sudo x11-utils xdg-user-dirs xdg-utils xserver-xorg-input-libinput-dev

apt install -y libprocps-dev libproc2-dev lxqt-qtplugin

#apt install -y openbox
#apt install -y sddm
#systemctl enable sddm 

echo "############install build-tools"
git clone https://github.com/lxqt/lxqt-build-tools.git
#wget https://github.com/lxqt/lxqt-build-tools/archive/refs/tags/0.13.0.tar.gz
#tar -xvf 0.13.0.tar.gz
cd lxqt-build-tools/
mkdir build && cd build
cmake ../ -DCMAKE_INSTALL_PREFIX=/usr
make && make install 
cd

git clone https://github.com/lxqt/lxqt.git
cd lxqt
git submodule init
git submodule update --remote --rebase

echo "########## building lxqt ########## "
LXQT_PREFIX=/usr ./build_all_cmake_projects.sh
echo "########## ALL WORK DONE.########## " 

echo "exec startlxqt" >> ~/.xinitrc
cp /usr/share/xsessions/lxqt.desktop /usr/share/
#startx 
#选择openbox 或者其他管理器
```
