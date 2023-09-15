# how to configure dwm

# ubuntu 开启root
```bash
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
``````

# dwm

## 官方git
```bash
基本的几件套 
git clone https://git.suckless.org/dmenu
cd dmenu && make && make install && cd

git clone https://git.suckless.org/st
cd st && sudo make clean install && cd


git clone git://git.suckless.org/dwm
cd dwm && make && make install && cd


官方有个状态栏
git clone git://git.suckless.org/dwmstatus
git clone git://git.suckless.org/slstatus

可以试试github上的dwmblocks
https://github.com/torrinfail/dwmblocks
https://github.com/LukeSmithxyz/dwmblocks

其他的状态监控
https://dwm.suckless.org/status_monitor/
```

## 其他小伙伴的github
```dotnetcli
https://github.com/gxt-kt/dwm
https://github.com/LukeSmithxyz/dwm.git
https://github.com/allinurl/dwm.git
https://github.com/diorgulescu/suckless-ubuntu/tree/master
https://github.com/yaocccc/dwm
https://github.com/Wjinlei/dwm
https://gitee.com/xiexie1993/dwm
https://github.com/bakkeby/dwm-flexipatch
``````

### dwm 简介
```bash
dwm 是动态窗口管理器。它能以平铺、单页和浮动布局管理窗口。所有布局都可以动态应用，并根据使用中的应用程序和执行的任务优化环境。

在平铺布局中，窗口由主区域和堆叠区域管理。主区域包含当前最需要关注的窗口，而堆叠区域则包含所有其他窗口。在单片式布局中，所有窗口都按屏幕大小最大化。在浮动布局中，窗口可以自由调整大小和移动。无论采用哪种布局，对话窗口始终都是浮动管理的。

窗口按标签分组。每个窗口都可以标记一个或多个标签。选择某些标签可显示带有这些标签的所有窗口。

每个屏幕都包含一个小的状态栏，显示所有可用的标签、布局、可见窗口的数量、聚焦窗口的标题，以及从根窗口名称属性读取的文本（如果屏幕被聚焦）。浮动窗口用空方框表示，最大化的浮动窗口在窗口标题前用填充方框表示。选定的标记用不同的颜色表示。聚焦窗口的标记在左上角用填充方块表示。应用于一个或多个窗口的标签在左上角用空方格表示

区别
与 ion、larswm 和 wmii 相比，dwm 更小、更快、更简单。
dwm 没有 Lua 集成，不支持 9P，没有基于 shell 的配置，没有远程控制，也没有任何附加工具，如打印选区或扭曲鼠标。
dwm 只有一个二进制文件，其源代码旨在保持小巧。
dwm 不区分图层：没有浮动图层或平铺图层。无论当前选定标签的客户端是否采用平铺布局，您都可以即时重新排列它们。不过，弹出窗口和固定大小窗口始终是浮动的。
dwm 可通过编辑源代码进行定制，这使得它运行速度极快，而且非常安全--除了从根窗口名称读取的窗口标题和状态文本外，它不会处理任何在编译时未知的输入数据。你不必学习 Lua/sh/ruby 或其他奇怪的配置文件格式（如 X 文件），也不必学习 C 语言，只需学习 C 语言（至少是为了编辑头文件），就能根据自己的需要定制 dwm。
因为 dwm 是通过编辑源代码定制的，所以制作二进制包毫无意义。这就使其用户群保持小规模和精英化。不会有新手提出愚蠢的问题。不过也有一些发行版提供二进制软件包。
dwm 读取根窗口的名称来打印任意状态文本（如日期、负载、电池电量）。这比 larsremote、wmiir 什么的简单多了...
dwm 为每个 Xinerama 屏幕创建一个视图。

如果您是第一次使用 dwm，请先阅读教程
https://dwm.suckless.org/tutorial/

定制化
dwm是通过编辑config.h（一个 C 语言头文件）和 config.mk（一个 Make include 文件）来定制的。
什么是config.h？
config.h 是一个源代码文件，包含在主要 dwm 源代码模块 dwm.c 中。它充当所有 dwm 功能的配置文件，例如应用程序放置、标签和颜色。dwm 的普通下载将包含一个名为 config.def.h 的文件，您可以使用该模板来创建自己的 config.h 文件。要开始自定义 dwm，只需在运行 make 之前将 config.def.h 复制到 config.h 中即可。
## cp config.def.h config.h

什么是config.mk？
config.mk 是 Makefile 包含的文件。它允许您配置 make 如何编译和安装 dwm。

如何修改config.h？
config.h 可以像任何其他 C 源代码文件一样进行编辑。它包含 dwm.c 将使用的变量的定义，因此该文件始终保持最新至关重要。使用 dwm 分发的默认 Makefile 不会用 config.def.h 的内容覆盖您自定义的 config.h，即使它在最新的 git pull 中进行了更新。因此，您应该始终将自定义的 config.h 与 config.def.h 进行比较，并确保在 config.h 中包含对后者的任何更改。

如何修改config.mk？
config.mk 可以像任何其他文本文件一样进行编辑。它包含将在 Makefile 中使用的变量的定义。与 config.h 不同，config.mk 没有 config.def.mk（默认 Makefile）。因此，在更新存储库期间，如果编辑原始 config.mk，则可能会遇到冲突。   

我们认为，Ion 或 wmi-10 中的静态窗口管理是一种过于死板和缺乏灵活性的工作环境。在 acme、larswm 和 oberon 中，动态窗口管理将用户从这些限制中解放出来。用户可以随心所欲地启动多个应用程序和窗口，并在窗口管理器的帮助下轻松地以有用的方式排列它们--工作环境随用户正在执行的任务而变化。这种体验非常流畅自然。wmii 和 dwm 也引入了类似的概念。

动态窗口管理规定，管理窗口是窗口管理器的工作，而不是用户的工作，用户必须设置一些只适用于特定工作场景的专门布局。长期以来，这一直是 larswm 的座右铭。与静态窗口管理相比，无论用户在做什么，也无论有多少应用程序在同时运行，他都很少需要考虑如何组织窗口。窗口管理器会适应当前的环境，帮助用户根据自己的需要进行管理和塑造，而不是强迫用户使用预设的固定布局，并试图将所有窗口和应用程序都塞入其中。

动态窗口管理有很多优点：你可以在几秒钟内创建和拆卸整个工作环境，而不是花时间去微调一个无法在所有情况下都能很好工作的固定布局。你所使用的窗口的数量和性质一直在变化，动态窗口管理器可以让你适应这种变化，并始终有效地利用宝贵的屏幕空间。
```

### 个人观点
```dotnetcli
主线版本仅提供基本的功能,like a baby
如果你想非常好用,只能通过定制,需要你的时间
不同人定制出来的dwm 区别非常大,在他的环境ok,你clone下来未必好用
官方项目有非常多的补丁, 且补丁的管理一言难尽,等同于没有管理

建议
1.熟练掌握git https://dwm.suckless.org/customisation/patches_in_git/
2.尽可能少的使用补丁文件 
3.可以试试该项目 https://github.com/bakkeby/dwm-flexipatch
```

### 安装依赖
```bash
#!/bin/bash

sudo apt install -y  suckless-tools libx11-dev libxft-dev libxinerama-dev gcc make build-essential cmake git
sudo apt update --fix-missing
sudo apt install -y libharfbuzz-dev
```
### 可能遇到的缺少库文件的解决办法
```bash
dwm.c:45:10: fatal error: X11/Xlib-xcb.h: No such file or directory
   45 | #include <X11/Xlib-xcb.h>
      |          ^~~~~~~~~~~~~~~~
compilation terminated.
make: *** [Makefile:18: dwm.o] Error 1
root@test:~/dwm#  apt install libxcb-res0-dev

__________________________________________________
$ sudo apt-get install apt-file
$ sudo apt-file update
$ apt-file search "X11/extensions/XTest.h"
libxtst-dev: /usr/include/X11/extensions/XTest.h
以此类推
```



### dwm 编译log
```log
root@test:~/dwm# make clean install
rm -f dwm drw.o dwm.o util.o dwm-6.4.tar.gz
dwm build options:
CFLAGS   = -std=c99 -pedantic -Wall -Wno-deprecated-declarations -Os -I/usr/X11R6/include -I/usr/include/freetype2 -D_DEFAULT_SOURCE -D_BSD_SOURCE -D_XOPEN_SOURCE=700L -DVERSION="6.4" -DXINERAMA
LDFLAGS  = -L/usr/X11R6/lib -lX11 -lXinerama -lfontconfig -lXft
CC       = cc
cc -c -std=c99 -pedantic -Wall -Wno-deprecated-declarations -Os -I/usr/X11R6/include -I/usr/include/freetype2 -D_DEFAULT_SOURCE -D_BSD_SOURCE -D_XOPEN_SOURCE=700L -DVERSION=\"6.4\" -DXINERAMA drw.c
cc -c -std=c99 -pedantic -Wall -Wno-deprecated-declarations -Os -I/usr/X11R6/include -I/usr/include/freetype2 -D_DEFAULT_SOURCE -D_BSD_SOURCE -D_XOPEN_SOURCE=700L -DVERSION=\"6.4\" -DXINERAMA dwm.c
cc -c -std=c99 -pedantic -Wall -Wno-deprecated-declarations -Os -I/usr/X11R6/include -I/usr/include/freetype2 -D_DEFAULT_SOURCE -D_BSD_SOURCE -D_XOPEN_SOURCE=700L -DVERSION=\"6.4\" -DXINERAMA util.c
cc -o dwm drw.o dwm.o util.o -L/usr/X11R6/lib -lX11 -lXinerama -lfontconfig -lXft
mkdir -p /usr/local/bin
cp -f dwm /usr/local/bin
chmod 755 /usr/local/bin/dwm
mkdir -p /usr/local/share/man/man1
sed "s/VERSION/6.4/g" < dwm.1 > /usr/local/share/man/man1/dwm.1
chmod 644 /usr/local/share/man/man1/dwm.1
root@test:~/dwm#
```

### 壁纸文件
```dotnetcli
git clone https://github.com/yaocccc/wallpaper.git
git clone https://gitlab.com/dwt1/wallpapers.git
```

### .xinitrc
```bash
feh --bg-fill "/home/test/Photos/Wallpapers/login.jpg"
        slstatus&
        exec dwm
``````

### dwm.desktop
```bash
## 可能需要
test@test:/usr/share/xsessions$ ls
dwm.desktop
test@test:/usr/share/xsessions$ cat dwm.desktop
[Desktop Entry]
Name=dwm
Comment=dynamic window manager
Exec=dwm
Icon=dwm
Type=Application
test@test:/usr/share/xsessions$
```

### dwm修改点-默认的工作模式的显示换成中文

```bash
vi config.h

/*line 42-44  把默认的工作模式的显示换成中文*/
 42         { "[]=",      tile },    /* first entry is default */
 43         { "><>",      NULL },    /* no layout function means floating behavior */
 44         { "[M]",      monocle },

 42         { "平铺模式",      tile },    /* first entry is default */
 43         { "浮动模式",      NULL },    /* no layout function means floating behavior */
 44         { "单窗口模式",      monocle },
```


### dwm修改点--把默认的alt功能键换成徽标键
```bash
/*line 48 把默认的alt功能键换成徽标键*/
#define MODKEY Mod1Mask
#define MODKEY Mod4Mask

```

### dwm修改点--super+enter 打开终端
```bash
/*line 66把默认的MODKEY+Shift+回车 打开终端变成MODKEY+回车打开终端*/
{ MODKEY|ShiftMask,   XK_Return, spawn,          {.v = termcmd } },
{ MODKEY,             XK_Return, spawn,          {.v = termcmd } },

/*line 73把默认的MODKEY+回车 窗口位置互换 变成MODKEY+ctrl+回车*/
{ MODKEY,                       XK_Return, zoom,           {0} },
{ MODKEY|ControlMask,              XK_Return, zoom,           {0} },


/*line 76把默认的MODKEY+Shift+c 关闭终端变成MODKEY+q关闭终端*/
{ MODKEY|ShiftMask,             XK_c,      killclient,     {0} },
{ MODKEY,                       XK_q,      killclient,     {0} },

/*line 68 69 把默认的super+j k 页面内切换窗口变为 super+tab 和super+tab切换窗口*/
{ MODKEY,                       XK_j,      focusstack,     {.i = +1 } },
{ MODKEY,                       XK_k,      focusstack,     {.i = -1 } },
{ MODKEY,              XK_Tab,          focusstack,       {.i = +1} },

/*line 96 把默认的super+shift+q退出dwm变为 super+Esc退出*/
{ MODKEY|ShiftMask,             XK_q,      quit,           {0} },
{ MODKEY,                  XK_Escape,      quit,           {0} },

```

### dwm 快捷键
| 默认快捷键                | 注释              | 修改后               |
|----------------------|-----------------|-------------------|
| Alt + shift + Enter  | # 打开新窗口         | super+Enter       |
| Alt + shift + c      | # 关闭当前窗口        | super+q           |
| Alt + D              | # 窗口横向排列        |                   |
| Alt + I              | # 窗口竖向排列        |                   |
| Alt + Enter          | # 窗口位置互换        | super+ctrl+Enter  |
| Alt + num            | # 切换标签页         |                   |
| Alt + shift + num    | # 移动窗口至某标签页  |                   |
| Alt + T              | # 平铺模式（tiling)  |                   |
| Alt + M              | # 单窗口模式         |                   |
| Alt + F              | # 浮动模式（float)   |                   |
| Alt + J              | # 在窗口间切换,向左   | super+Tab         |
| Alt + K              | # 在窗口间切换,向右   |                   |
| Alt + H              | # 改变窗口的长度/比例 |                   |
| Alt + L              |                     |                   |
| Alt + Space          | # 窗口模式切换        | super+Space       |
| Alt + shift + Space  |                     |                   |
| Mod + <              | 主副屏之间移动焦点至左边屏幕  |                   |
| Mod + >              | 主副屏之间移动焦点至右边屏幕  |                   |
| Mod + shift + <      | 在主副屏之间移动窗口至左边屏幕 |                   |
| Mod + shift + >      | 在主副屏之间移动窗口至右边屏幕 |                   |
| Mod + 鼠标右键           | 缩放窗口                |                   |
| Mod+p                | dmenu           | supet+p           |

## st部分
[参考阅读](https://wiki.archlinuxcn.org/wiki/St)

### st编译log
```bash
root@test:~/st# make clean install
rm -f st st.o x.o st-0.9.tar.gz
c99 -I/usr/X11R6/include  `pkg-config --cflags fontconfig`  `pkg-config --cflags freetype2` -DVERSION=\"0.9\" -D_XOPEN_SOURCE=600  -O1 -c st.c
c99 -I/usr/X11R6/include  `pkg-config --cflags fontconfig`  `pkg-config --cflags freetype2` -DVERSION=\"0.9\" -D_XOPEN_SOURCE=600  -O1 -c x.c
c99 -o st st.o x.o -L/usr/X11R6/lib -lm -lrt -lX11 -lutil -lXft  `pkg-config --libs fontconfig`  `pkg-config --libs freetype2`
mkdir -p /usr/local/bin
cp -f st /usr/local/bin
chmod 755 /usr/local/bin/st
mkdir -p /usr/local/share/man/man1
sed "s/VERSION/0.9/g" < st.1 > /usr/local/share/man/man1/st.1
chmod 644 /usr/local/share/man/man1/st.1
tic -sx st.info
7 entries written to /etc/terminfo
Please see the README file regarding the terminfo entry of st.
root@test:~/st#
```
### 安装卸载补丁
```dotnetcli
安装补丁包：patch  -p(n)  < [补丁包路径] patch_name
卸载补丁包：patch  -p(n)  -R  < [补丁包路径] patch_name
下面简要解释参数n的意义：
-p(n)中的n表示将patch中的路径第N条‘/’和其左边部分取消。

patch -p1 < st-alpha-20220206-0.8.5.diff   #安装补丁
patch -p1 -R < st-alpha-20220206-0.8.5.diff  #卸载补丁

好的补丁管理方式是git ,每个补丁建立一个分支, 逐个合入主分支
但补丁越来越多,冲突在所难免
推荐工具 kdiff3 

```

### scroll 日志翻页功能-可选
```dotnetcli
root@test:~# git clone git://git.suckless.org/scroll
...
root@test:~# cd scroll/
root@test:~/scroll# ls
LICENSE  Makefile  README  TODO  config.def.h  config.mk  perf.sh  ptty.c  scroll.1  scroll.c  up.log  up.sh
root@test:~/scroll# make clean install

然后修改st/config.h line 22 ,让st支持scroll的翻页功能
scroll目前仍处于试验性质

char *scroll = "/usr/local/bin/scroll";

该种方法对st本身的修改比较小,且回退简单
```

### st-透明效果补丁
```dotnetcli
st-alpha-0.8.5.diff

root@test:~/st# patch -p1 < st-alpha-20220206-0.8.5.diff
patching file config.def.h
patching file config.mk
patching file st.h
Hunk #1 succeeded at 124 (offset -2 lines).
patching file x.c
Hunk #3 succeeded at 754 (offset 16 lines).
Hunk #4 succeeded at 814 (offset 16 lines).
Hunk #5 succeeded at 1143 (offset 16 lines).
Hunk #6 succeeded at 1169 (offset 16 lines).
Hunk #7 succeeded at 1189 (offset 16 lines).
Hunk #8 succeeded at 2055 (offset 19 lines).

在config.h 中设置透明度
line 97 float alpha = 0.1;

root@test:~/st# make clean install
rm -f st st.o x.o st-0.9.tar.gz
cp config.def.h config.h
c99 -I/usr/X11R6/include  `pkg-config --cflags fontconfig`  `pkg-config --cflags freetype2` -DVERSION=\"0.9\" -D_XOPEN_SOURCE=600  -O1 -c st.c
c99 -I/usr/X11R6/include  `pkg-config --cflags fontconfig`  `pkg-config --cflags freetype2` -DVERSION=\"0.9\" -D_XOPEN_SOURCE=600  -O1 -c x.c
c99 -o st st.o x.o -L/usr/X11R6/lib -lm -lrt -lX11 -lutil -lXft -lXrender `pkg-config --libs fontconfig`  `pkg-config --libs freetype2`
mkdir -p /usr/local/bin
cp -f st /usr/local/bin
chmod 755 /usr/local/bin/st
mkdir -p /usr/local/share/man/man1
sed "s/VERSION/0.9/g" < st.1 > /usr/local/share/man/man1/st.1
chmod 644 /usr/local/share/man/man1/st.1
tic -sx st.info
7 entries written to /etc/terminfo
Please see the README file regarding the terminfo entry of st.
root@test:~/st#    
```

### 透明效果补丁先安装picom
```dotnetcli
设置为picom 后台运行
root@test:~/st# cat /root/.xinitrc 
feh --bg-fill --randomize /root/wallpaper/*
picom -b
exec dwm
root@test:~/st#

透明功能需要一个X composite manager（例如picom、compton、 xcompmgr)
```

### st-终端占满全屏补丁
```dotnetcli
st-anysize-20220718-baa9357.diff

root@test:~/st# patch < st-anysize-20220718-baa9357.diff
patching file x.c
Hunk #2 succeeded at 334 (offset 2 lines).
Hunk #3 succeeded at 342 (offset 2 lines).
Hunk #4 succeeded at 742 (offset 2 lines).
Hunk #5 succeeded at 882 (offset 9 lines).
Hunk #6 succeeded at 1177 (offset 21 lines).
Hunk #7 succeeded at 1263 (offset 17 lines).
Hunk #8 succeeded at 1396 (offset 17 lines).
Hunk #9 succeeded at 1486 (offset 17 lines).
Hunk #10 succeeded at 1590 (offset 17 lines).
root@test:~/st# make clean install
rm -f st st.o x.o st-0.9.tar.gz
c99 -I/usr/X11R6/include  `pkg-config --cflags fontconfig`  `pkg-config --cflags freetype2` -DVERSION=\"0.9\" -D_XOPEN_SOURCE=600  -O1 -c st.c
c99 -I/usr/X11R6/include  `pkg-config --cflags fontconfig`  `pkg-config --cflags freetype2` -DVERSION=\"0.9\" -D_XOPEN_SOURCE=600  -O1 -c x.c
c99 -o st st.o x.o -L/usr/X11R6/lib -lm -lrt -lX11 -lutil -lXft -lXrender `pkg-config --libs fontconfig`  `pkg-config --libs freetype2`
mkdir -p /usr/local/bin
cp -f st /usr/local/bin
chmod 755 /usr/local/bin/st
mkdir -p /usr/local/share/man/man1
sed "s/VERSION/0.9/g" < st.1 > /usr/local/share/man/man1/st.1
chmod 644 /usr/local/share/man/man1/st.1
tic -sx st.info
7 entries written to /etc/terminfo
Please see the README file regarding the terminfo entry of st.
root@test:~/st#

```

### st-日志屏幕回滚补丁
```dotnetcli
https://st.suckless.org/patches/scrollback/ 
这个补丁比较复杂,补丁之间有依赖关系
root@test:~/st# patch < st-scrollback-ringbuffer-0.8.5.diff
patching file config.def.h
Hunk #1 succeeded at 204 (offset 3 lines).
patching file st.c
Hunk #3 succeeded at 216 (offset 1 line).
Hunk #4 succeeded at 423 (offset -7 lines).
Hunk #5 succeeded at 537 (offset -7 lines).
Hunk #6 succeeded at 552 (offset -7 lines).
Hunk #7 succeeded at 580 (offset -7 lines).
Hunk #8 succeeded at 618 (offset -7 lines).
Hunk #9 succeeded at 965 (offset -7 lines).
Hunk #10 succeeded at 995 (offset -7 lines).
Hunk #11 succeeded at 1018 (offset -7 lines).
Hunk #12 succeeded at 1042 (offset -7 lines).
Hunk #13 succeeded at 1080 (offset -7 lines).
Hunk #14 succeeded at 1124 (offset -7 lines).
Hunk #15 succeeded at 1158 (offset -7 lines).
Hunk #16 succeeded at 1284 (offset -7 lines).
Hunk #17 succeeded at 1293 (offset -7 lines).
Hunk #18 succeeded at 1319 (offset -7 lines).
Hunk #19 succeeded at 1336 (offset -7 lines).
Hunk #20 succeeded at 1351 (offset -7 lines).
Hunk #21 succeeded at 1368 (offset -7 lines).
Hunk #22 succeeded at 2171 (offset -25 lines).
Hunk #23 succeeded at 2557 with fuzz 1 (offset -22 lines).
Hunk #24 succeeded at 2571 (offset -20 lines).
Hunk #25 succeeded at 2602 (offset -20 lines).
Hunk #26 succeeded at 2633 (offset -20 lines).
Hunk #27 succeeded at 2721 (offset -20 lines).
Hunk #28 succeeded at 2742 (offset -20 lines).
Hunk #29 succeeded at 2765 (offset -20 lines).
patching file st.h
patching file x.c
root@test:~/st# 
root@test:~/st# patch < st-scrollback-mouse-20220127-2c5edf2.diff
patching file config.def.h
Hunk #1 succeeded at 179 (offset 3 lines).
root@test:~/st# ls
FAQ     LICENSE   README  arg.h         config.def.h.orig  config.mk  st-alpha-20220206-0.8.5.diff      st-scrollback-mouse-20220127-2c5edf2.diff  st.1  st.c.orig  st.h.orig  st.o   x.c       x.o
LEGACY  Makefile  TODO    config.def.h  config.h           st         st-anysize-20220718-baa9357.diff  st-scrollback-ringbuffer-0.8.5.diff        st.c  st.h       st.info    win.h  x.c.orig
root@test:~/st# rm config.h
root@test:~/st# make clean install
rm -f st st.o x.o st-0.9.tar.gz
cp config.def.h config.h
c99 -I/usr/X11R6/include  `pkg-config --cflags fontconfig`  `pkg-config --cflags freetype2` -DVERSION=\"0.9\" -D_XOPEN_SOURCE=600  -O1 -c st.c
c99 -I/usr/X11R6/include  `pkg-config --cflags fontconfig`  `pkg-config --cflags freetype2` -DVERSION=\"0.9\" -D_XOPEN_SOURCE=600  -O1 -c x.c
c99 -o st st.o x.o -L/usr/X11R6/lib -lm -lrt -lX11 -lutil -lXft -lXrender `pkg-config --libs fontconfig`  `pkg-config --libs freetype2`
mkdir -p /usr/local/bin
cp -f st /usr/local/bin
chmod 755 /usr/local/bin/st
mkdir -p /usr/local/share/man/man1
sed "s/VERSION/0.9/g" < st.1 > /usr/local/share/man/man1/st.1
chmod 644 /usr/local/share/man/man1/st.1
tic -sx st.info
7 entries written to /etc/terminfo
Please see the README file regarding the terminfo entry of st.
root@test:~/st#

```

### 光标颜色
```dotnetcli
config.h line 121 ##打的补丁不一样的话,行数可能不一样
"#cccccc",
改为
"green3",
```

简单举例至此

