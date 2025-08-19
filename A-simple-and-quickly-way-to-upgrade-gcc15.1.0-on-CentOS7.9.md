# 1. A simple and quickly way to upgrade gcc15.1.0 on CentOS7.9

No complicated compilation process,
no complicated dependencies,
simple and fast.

## 1.1 Environment Introduction
```dotnetcli
[root@localhost ~]# cat /etc/os-release
NAME="CentOS Linux"
VERSION="7 (Core)"
ID="centos"
ID_LIKE="rhel fedora"
VERSION_ID="7"
PRETTY_NAME="CentOS Linux 7 (Core)"
ANSI_COLOR="0;31"
CPE_NAME="cpe:/o:centos:centos:7"
HOME_URL="https://www.centos.org/"
BUG_REPORT_URL="https://bugs.centos.org/"

CENTOS_MANTISBT_PROJECT="CentOS-7"
CENTOS_MANTISBT_PROJECT_VERSION="7"
REDHAT_SUPPORT_PRODUCT="centos"
REDHAT_SUPPORT_PRODUCT_VERSION="7"


[root@localhost ~]# echo $0
-bash
[root@localhost ~]#
[root@localhost ~]# bash --version
GNU bash, version 4.2.46(2)-release (x86_64-redhat-linux-gnu)
Copyright (C) 2011 Free Software Foundation, Inc.
License GPLv3+: GNU GPL version 3 or later <http://gnu.org/licenses/gpl.html>

This is free software; you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.
[root@localhost ~]#
[root@localhost ~]# gcc -v
使用内建 specs。
COLLECT_GCC=gcc
COLLECT_LTO_WRAPPER=/usr/libexec/gcc/x86_64-redhat-linux/4.8.5/lto-wrapper
目标：x86_64-redhat-linux
配置为：../configure --prefix=/usr --mandir=/usr/share/man --infodir=/usr/share/info --with-bugurl=http://bugzilla.redhat.com/bugzilla --enable-bootstrap --enable-shared --enable-threads=posix --enable-checking=release --with-system-zlib --enable-__cxa_atexit --disable-libunwind-exceptions --enable-gnu-unique-object --enable-linker-build-id --with-linker-hash-style=gnu --enable-languages=c,c++,objc,obj-c++,java,fortran,ada,go,lto --enable-plugin --enable-initfini-array --disable-libgcj --with-isl=/builddir/build/BUILD/gcc-4.8.5-20150702/obj-x86_64-redhat-linux/isl-install --with-cloog=/builddir/build/BUILD/gcc-4.8.5-20150702/obj-x86_64-redhat-linux/cloog-install --enable-gnu-indirect-function --with-tune=generic --with-arch_32=x86-64 --build=x86_64-redhat-linux
线程模型：posix
gcc 版本 4.8.5 20150623 (Red Hat 4.8.5-44) (GCC)
[root@localhost ~]#
```


## 1.2 Install cmake 4.1.0
```dotnetcli
wget https://github.com/Kitware/CMake/releases/download/v4.1.0/cmake-4.1.0-linux-x86_64.sh
chmod +x cmake-4.1.0-linux-x86_64.sh
./cmake-4.1.0-linux-x86_64.sh
accept the license
```

## 1.3 Install xlings

```dotnetcli
curl -fsSL https://d2learn.org/xlings-install.sh | bash

```

## 1.4 Install gcc 15.1.0
```dotnetcli
xlings install gcc@15
```
log
```dotnetcli
[root@localhost ~]# xlings install gcc@15.1.0

	**Warning: don't recommend run xlings as root**

[xlings:xim]: create pm executor for <gcc@15.1.0> ...

--- [package] info

name: gcc
version: 15.1.0
authors: GNU
licenses: GPL
repo: https://github.com/gcc-mirror/gcc
docs: https://gcc.gnu.org/wiki
programs: gcc-static, g++-static, gcc, g++, c++, cpp, addr2line, ar, as, ld, nm, objcopy, objdump, ranlib, readelf, size, strings, strip, ldd, loader

	GCC, the GNU Compiler Collection

-> install gcc@15.1.0? (y/n) y
[xlings:xim]: checking [gcc@15.1.0] for mutex groups...
[xlings:xim]: skip download (url is nil)
[xlings:xim]: start install gcc, it may take some minutes...
[xlings:xim]: create install dir /home/xlings/.xlings_data/xim/xpkgs/gcc/15.1.0
[xlings:xim]: create pm executor for <musl-gcc@15.1.0> ...
[xlings:xim]: checking [musl-gcc@15.1.0] for mutex groups...
[xlings]: downloading: https://gitcode.com/xlings-res/musl-gcc/releases/download/15.1.0/musl-gcc-15.1.0-linux-x86_64.tar.gz to /home/xlings/.xlings_data/xim/runtimedir/musl-gcc-15.1.0-linux-x86_64.tar.gz
######################################################################## 100.0%
[xlings:xim]: start extract musl-gcc-15.1.0-linux-x86_64.tar.gz
[xlings:xim]: start install musl-gcc, it may take some minutes...
[xlings:xim]: create install dir /home/xlings/.xlings_data/xim/xpkgs/musl-gcc/15.1.0
[xlings:xim]: start config...
[xlings:xim]: xvm run - /home/xlings/.xlings_data/bin/xvm add musl-gcc 15.1.0 --path /home/xlings/.xlings_data/xim/xpkgs/musl-gcc/15.1.0/bin    --alias "x86_64-linux-musl-gcc"
adding target: musl-gcc, version: 15.1.0
set [musl-gcc 15.1.0] as default
[xlings:xim]: xvm run - /home/xlings/.xlings_data/bin/xvm add musl-g++ 15.1.0 --path /home/xlings/.xlings_data/xim/xpkgs/musl-gcc/15.1.0/bin    --alias "x86_64-linux-musl-g++"
adding target: musl-g++, version: 15.1.0
set [musl-g++ 15.1.0] as default
[xlings:xim]: xvm run - /home/xlings/.xlings_data/bin/xvm add musl-c++ 15.1.0 --path /home/xlings/.xlings_data/xim/xpkgs/musl-gcc/15.1.0/bin    --alias "x86_64-linux-musl-c++"
adding target: musl-c++, version: 15.1.0
set [musl-c++ 15.1.0] as default
[xlings:xim]: xvm run - /home/xlings/.xlings_data/bin/xvm add musl-cpp 15.1.0 --path /home/xlings/.xlings_data/xim/xpkgs/musl-gcc/15.1.0/bin    --alias "x86_64-linux-musl-cpp"
adding target: musl-cpp, version: 15.1.0
set [musl-cpp 15.1.0] as default
[xlings:xim]: xvm run - /home/xlings/.xlings_data/bin/xvm add musl-addr2line 15.1.0 --path /home/xlings/.xlings_data/xim/xpkgs/musl-gcc/15.1.0/bin    --alias "x86_64-linux-musl-addr2line"
adding target: musl-addr2line, version: 15.1.0
set [musl-addr2line 15.1.0] as default
[xlings:xim]: xvm run - /home/xlings/.xlings_data/bin/xvm add musl-ar 15.1.0 --path /home/xlings/.xlings_data/xim/xpkgs/musl-gcc/15.1.0/bin    --alias "x86_64-linux-musl-ar"
adding target: musl-ar, version: 15.1.0
set [musl-ar 15.1.0] as default
[xlings:xim]: xvm run - /home/xlings/.xlings_data/bin/xvm add musl-as 15.1.0 --path /home/xlings/.xlings_data/xim/xpkgs/musl-gcc/15.1.0/bin    --alias "x86_64-linux-musl-as"
adding target: musl-as, version: 15.1.0
set [musl-as 15.1.0] as default
[xlings:xim]: xvm run - /home/xlings/.xlings_data/bin/xvm add musl-ld 15.1.0 --path /home/xlings/.xlings_data/xim/xpkgs/musl-gcc/15.1.0/bin    --alias "x86_64-linux-musl-ld"
adding target: musl-ld, version: 15.1.0
set [musl-ld 15.1.0] as default
[xlings:xim]: xvm run - /home/xlings/.xlings_data/bin/xvm add musl-nm 15.1.0 --path /home/xlings/.xlings_data/xim/xpkgs/musl-gcc/15.1.0/bin    --alias "x86_64-linux-musl-nm"
adding target: musl-nm, version: 15.1.0
set [musl-nm 15.1.0] as default
[xlings:xim]: xvm run - /home/xlings/.xlings_data/bin/xvm add musl-objcopy 15.1.0 --path /home/xlings/.xlings_data/xim/xpkgs/musl-gcc/15.1.0/bin    --alias "x86_64-linux-musl-objcopy"
adding target: musl-objcopy, version: 15.1.0
set [musl-objcopy 15.1.0] as default
[xlings:xim]: xvm run - /home/xlings/.xlings_data/bin/xvm add musl-objdump 15.1.0 --path /home/xlings/.xlings_data/xim/xpkgs/musl-gcc/15.1.0/bin    --alias "x86_64-linux-musl-objdump"
adding target: musl-objdump, version: 15.1.0
set [musl-objdump 15.1.0] as default
[xlings:xim]: xvm run - /home/xlings/.xlings_data/bin/xvm add musl-ranlib 15.1.0 --path /home/xlings/.xlings_data/xim/xpkgs/musl-gcc/15.1.0/bin    --alias "x86_64-linux-musl-ranlib"
adding target: musl-ranlib, version: 15.1.0
set [musl-ranlib 15.1.0] as default
[xlings:xim]: xvm run - /home/xlings/.xlings_data/bin/xvm add musl-readelf 15.1.0 --path /home/xlings/.xlings_data/xim/xpkgs/musl-gcc/15.1.0/bin    --alias "x86_64-linux-musl-readelf"
adding target: musl-readelf, version: 15.1.0
set [musl-readelf 15.1.0] as default
[xlings:xim]: xvm run - /home/xlings/.xlings_data/bin/xvm add musl-size 15.1.0 --path /home/xlings/.xlings_data/xim/xpkgs/musl-gcc/15.1.0/bin    --alias "x86_64-linux-musl-size"
adding target: musl-size, version: 15.1.0
set [musl-size 15.1.0] as default
[xlings:xim]: xvm run - /home/xlings/.xlings_data/bin/xvm add musl-strings 15.1.0 --path /home/xlings/.xlings_data/xim/xpkgs/musl-gcc/15.1.0/bin    --alias "x86_64-linux-musl-strings"
adding target: musl-strings, version: 15.1.0
set [musl-strings 15.1.0] as default
[xlings:xim]: xvm run - /home/xlings/.xlings_data/bin/xvm add musl-strip 15.1.0 --path /home/xlings/.xlings_data/xim/xpkgs/musl-gcc/15.1.0/bin    --alias "x86_64-linux-musl-strip"
adding target: musl-strip, version: 15.1.0
set [musl-strip 15.1.0] as default
[xim:xpkg]: add runtime libraries for musl-gcc-static...
[xlings:xim]: xvm run - /home/xlings/.xlings_data/bin/xvm add musl-libc musl-gcc-15.1.0 --path /home/xlings/.xlings_data/xim/xpkgs/musl-gcc/15.1.0/x86_64-linux-musl/lib  --type "lib"  --filename "libc.so"  --alias "libc.so"
adding target: musl-libc, version: musl-gcc-15.1.0
set [musl-libc musl-gcc-15.1.0] as default
link [musl-libc musl-gcc-15.1.0] to [/home/xlings/.xlings_data/lib] ...
[xlings:xim]: xvm run - /home/xlings/.xlings_data/bin/xvm add libstdc++ musl-gcc-15.1.0 --path /home/xlings/.xlings_data/xim/xpkgs/musl-gcc/15.1.0/x86_64-linux-musl/lib  --type "lib"  --filename "libstdc++.so.6"  --alias "libstdc++.so.6"
adding target: libstdc++, version: musl-gcc-15.1.0
set [libstdc++ musl-gcc-15.1.0] as default
link [libstdc++ musl-gcc-15.1.0] to [/home/xlings/.xlings_data/lib] ...
[xlings:xim]: xvm run - /home/xlings/.xlings_data/bin/xvm add libgcc_s musl-gcc-15.1.0 --path /home/xlings/.xlings_data/xim/xpkgs/musl-gcc/15.1.0/x86_64-linux-musl/lib  --type "lib"  --filename "libgcc_s.so.1"  --alias "libgcc_s.so.1"
adding target: libgcc_s, version: musl-gcc-15.1.0
set [libgcc_s musl-gcc-15.1.0] as default
link [libgcc_s musl-gcc-15.1.0] to [/home/xlings/.xlings_data/lib] ...
[xlings:xim]: xvm run - /home/xlings/.xlings_data/bin/xvm add ld-musl musl-gcc-15.1.0 --path /home/xlings/.xlings_data/xim/xpkgs/musl-gcc/15.1.0/x86_64-linux-musl/lib  --type "lib"  --filename "ld-musl-x86_64.so.1"  --alias "libc.so"
adding target: ld-musl, version: musl-gcc-15.1.0
set [ld-musl musl-gcc-15.1.0] as default
link [ld-musl musl-gcc-15.1.0] to [/home/xlings/.xlings_data/lib] ...
[xlings:xim]: xvm run - /home/xlings/.xlings_data/bin/xvm add musl-ldd musl-gcc-15.1.0 --path /home/xlings/.xlings_data/xim/xpkgs/musl-gcc/15.1.0/x86_64-linux-musl/lib    --alias "libc.so --list"  --env "LD_LIBRARY_PATH=/home/xlings/.xlings_data/xim/xpkgs/musl-gcc/15.1.0/x86_64-linux-musl/lib"
adding target: musl-ldd, version: musl-gcc-15.1.0
set [musl-ldd musl-gcc-15.1.0] as default
[xlings:xim]: xvm run - /home/xlings/.xlings_data/bin/xvm add musl-loader musl-gcc-15.1.0 --path /home/xlings/.xlings_data/xim/xpkgs/musl-gcc/15.1.0/x86_64-linux-musl/lib    --alias "libc.so"  --env "LD_LIBRARY_PATH=/home/xlings/.xlings_data/xim/xpkgs/musl-gcc/15.1.0/x86_64-linux-musl/lib"
adding target: musl-loader, version: musl-gcc-15.1.0
set [musl-loader musl-gcc-15.1.0] as default
[xim:xpkg]: add static wrapper for musl-gcc ...
[xlings:xim]: xvm run - /home/xlings/.xlings_data/bin/xvm add musl-gcc-static 15.1.0 --path /home/xlings/.xlings_data/xim/xpkgs/musl-gcc/15.1.0    --alias "musl-gcc -static"
adding target: musl-gcc-static, version: 15.1.0
set [musl-gcc-static 15.1.0] as default
[xlings:xim]: xvm run - /home/xlings/.xlings_data/bin/xvm add musl-g++-static 15.1.0 --path /home/xlings/.xlings_data/xim/xpkgs/musl-gcc/15.1.0    --alias "musl-g++ -static"
adding target: musl-g++-static, version: 15.1.0
set [musl-g++-static 15.1.0] as default
[xlings:xim]: musl-gcc@15.1.0 - installed
[xlings:xim]: update index database
[xlings:xim]: start config...
[xim:xpkg]: add [ gcc, g++, c++, cpp, addr2line, ar, as, ld, nm, objcopy, objdump, ranlib, readelf, size, strings,strip ... ] commands
[xlings:xim]: xvm run - /home/xlings/.xlings_data/bin/xvm add gcc-static 15.1.0 --path /home/xlings/.xlings_data/xim/xpkgs/gcc/15.1.0    --alias "musl-gcc-static"
adding target: gcc-static, version: 15.1.0
set [gcc-static 15.1.0] as default
[xlings:xim]: xvm run - /home/xlings/.xlings_data/bin/xvm add g++-static 15.1.0 --path /home/xlings/.xlings_data/xim/xpkgs/gcc/15.1.0    --alias "musl-g++-static"
adding target: g++-static, version: 15.1.0
set [g++-static 15.1.0] as default
[xlings:xim]: xvm run - /home/xlings/.xlings_data/bin/xvm add gcc 15.1.0 --path /home/xlings/.xlings_data/xim/xpkgs/gcc/15.1.0    --alias "musl-gcc"
adding target: gcc, version: 15.1.0
set [gcc 15.1.0] as default
[xlings:xim]: xvm run - /home/xlings/.xlings_data/bin/xvm add g++ 15.1.0 --path /home/xlings/.xlings_data/xim/xpkgs/gcc/15.1.0    --alias "musl-g++"
adding target: g++, version: 15.1.0
set [g++ 15.1.0] as default
[xlings:xim]: xvm run - /home/xlings/.xlings_data/bin/xvm add c++ 15.1.0 --path /home/xlings/.xlings_data/xim/xpkgs/gcc/15.1.0    --alias "musl-c++"
adding target: c++, version: 15.1.0
set [c++ 15.1.0] as default
[xlings:xim]: xvm run - /home/xlings/.xlings_data/bin/xvm add cpp 15.1.0 --path /home/xlings/.xlings_data/xim/xpkgs/gcc/15.1.0    --alias "musl-cpp"
adding target: cpp, version: 15.1.0
set [cpp 15.1.0] as default
[xlings:xim]: xvm run - /home/xlings/.xlings_data/bin/xvm add addr2line 15.1.0 --path /home/xlings/.xlings_data/xim/xpkgs/gcc/15.1.0    --alias "musl-addr2line"
adding target: addr2line, version: 15.1.0
set [addr2line 15.1.0] as default
[xlings:xim]: xvm run - /home/xlings/.xlings_data/bin/xvm add ar 15.1.0 --path /home/xlings/.xlings_data/xim/xpkgs/gcc/15.1.0    --alias "musl-ar"
adding target: ar, version: 15.1.0
set [ar 15.1.0] as default
[xlings:xim]: xvm run - /home/xlings/.xlings_data/bin/xvm add as 15.1.0 --path /home/xlings/.xlings_data/xim/xpkgs/gcc/15.1.0    --alias "musl-as"
adding target: as, version: 15.1.0
set [as 15.1.0] as default
[xlings:xim]: xvm run - /home/xlings/.xlings_data/bin/xvm add ld 15.1.0 --path /home/xlings/.xlings_data/xim/xpkgs/gcc/15.1.0    --alias "musl-ld"
adding target: ld, version: 15.1.0
set [ld 15.1.0] as default
[xlings:xim]: xvm run - /home/xlings/.xlings_data/bin/xvm add nm 15.1.0 --path /home/xlings/.xlings_data/xim/xpkgs/gcc/15.1.0    --alias "musl-nm"
adding target: nm, version: 15.1.0
set [nm 15.1.0] as default
[xlings:xim]: xvm run - /home/xlings/.xlings_data/bin/xvm add objcopy 15.1.0 --path /home/xlings/.xlings_data/xim/xpkgs/gcc/15.1.0    --alias "musl-objcopy"
adding target: objcopy, version: 15.1.0
set [objcopy 15.1.0] as default
[xlings:xim]: xvm run - /home/xlings/.xlings_data/bin/xvm add objdump 15.1.0 --path /home/xlings/.xlings_data/xim/xpkgs/gcc/15.1.0    --alias "musl-objdump"
adding target: objdump, version: 15.1.0
set [objdump 15.1.0] as default
[xlings:xim]: xvm run - /home/xlings/.xlings_data/bin/xvm add ranlib 15.1.0 --path /home/xlings/.xlings_data/xim/xpkgs/gcc/15.1.0    --alias "musl-ranlib"
adding target: ranlib, version: 15.1.0
set [ranlib 15.1.0] as default
[xlings:xim]: xvm run - /home/xlings/.xlings_data/bin/xvm add readelf 15.1.0 --path /home/xlings/.xlings_data/xim/xpkgs/gcc/15.1.0    --alias "musl-readelf"
adding target: readelf, version: 15.1.0
set [readelf 15.1.0] as default
[xlings:xim]: xvm run - /home/xlings/.xlings_data/bin/xvm add size 15.1.0 --path /home/xlings/.xlings_data/xim/xpkgs/gcc/15.1.0    --alias "musl-size"
adding target: size, version: 15.1.0
set [size 15.1.0] as default
[xlings:xim]: xvm run - /home/xlings/.xlings_data/bin/xvm add strings 15.1.0 --path /home/xlings/.xlings_data/xim/xpkgs/gcc/15.1.0    --alias "musl-strings"
adding target: strings, version: 15.1.0
set [strings 15.1.0] as default
[xlings:xim]: xvm run - /home/xlings/.xlings_data/bin/xvm add strip 15.1.0 --path /home/xlings/.xlings_data/xim/xpkgs/gcc/15.1.0    --alias "musl-strip"
adding target: strip, version: 15.1.0
set [strip 15.1.0] as default
[xlings:xim]: xvm run - /home/xlings/.xlings_data/bin/xvm add ldd 15.1.0 --path /home/xlings/.xlings_data/xim/xpkgs/gcc/15.1.0    --alias "musl-ldd"
adding target: ldd, version: 15.1.0
set [ldd 15.1.0] as default
[xlings:xim]: xvm run - /home/xlings/.xlings_data/bin/xvm add loader 15.1.0 --path /home/xlings/.xlings_data/xim/xpkgs/gcc/15.1.0    --alias "musl-loader"
adding target: loader, version: 15.1.0
set [loader 15.1.0] as default

	 **maybe need to restart cmd/shell to load env**
	       try to run source ~/.bashrc

[xlings:xim]: gcc@15.1.0 - installed

	     反馈 & 交流 | Feedback & Discourse
	(if encounter any problem, please report it)

	https://forum.d2learn.org/category/9/xlings
	https://github.com/d2learn/xlings/issues

[xlings:xim]: update index database
[root@localhost ~]# gcc -v
bash: gcc: 未找到命令...
[root@localhost ~]# cd /home/xlings/.xlings
.xlings/      .xlings_data/
[root@localhost ~]# cd /home/xlings/.xlings_data/xim/xpkgs/gcc/15.1.0/
[root@localhost 15.1.0]# ls
[root@localhost 15.1.0]# ls -al
总用量 0
drwxr-sr-x 2 root xlings  6 8月  18 09:39 .
drwxr-sr-x 3 root xlings 20 8月  18 09:39 ..
[root@localhost 15.1.0]# source ~/.bashrc
[root@localhost 15.1.0]# gcc -v
Reading specs from /home/xlings/.xlings_data/xim/xpkgs/musl-gcc/15.1.0/bin/../lib/gcc/x86_64-linux-musl/15.1.0/specs
COLLECT_GCC=x86_64-linux-musl-gcc
COLLECT_LTO_WRAPPER=/home/xlings/.xlings_data/xim/xpkgs/musl-gcc/15.1.0/bin/../libexec/gcc/x86_64-linux-musl/15.1.0/lto-wrapper
Target: x86_64-linux-musl
Configured with: ../src_gcc/configure --enable-languages=c,c++ --with-specs='%{!static:%{!shared:%{!static-pie: -Wl,--enable-new-dtags -Wl,-rpath,/home/xlings/.xlings_data/lib }}}' --disable-bootstrap --disable-assembly --disable-werror --target=x86_64-linux-musl --prefix= --libdir=/lib --disable-multilib --with-sysroot=/x86_64-linux-musl --enable-tls --disable-libmudflap --disable-libsanitizer --disable-gnu-indirect-function --disable-libmpx --enable-initfini-array --enable-libstdcxx-time=rt --with-build-sysroot=/home/xlings/.xlings_data/xim/xpkgs/musl-cross-make/0.0.1/musl-cross-make/build/local/x86_64-linux-musl/obj_sysroot AR_FOR_TARGET=/home/xlings/.xlings_data/xim/xpkgs/musl-cross-make/0.0.1/musl-cross-make/build/local/x86_64-linux-musl/obj_binutils/binutils/ar AS_FOR_TARGET=/home/xlings/.xlings_data/xim/xpkgs/musl-cross-make/0.0.1/musl-cross-make/build/local/x86_64-linux-musl/obj_binutils/gas/as-newLD_FOR_TARGET=/home/xlings/.xlings_data/xim/xpkgs/musl-cross-make/0.0.1/musl-cross-make/build/local/x86_64-linux-musl/obj_binutils/ld/ld-new NM_FOR_TARGET=/home/xlings/.xlings_data/xim/xpkgs/musl-cross-make/0.0.1/musl-cross-make/build/local/x86_64-linux-musl/obj_binutils/binutils/nm-new OBJCOPY_FOR_TARGET=/home/xlings/.xlings_data/xim/xpkgs/musl-cross-make/0.0.1/musl-cross-make/build/local/x86_64-linux-musl/obj_binutils/binutils/objcopy OBJDUMP_FOR_TARGET=/home/xlings/.xlings_data/xim/xpkgs/musl-cross-make/0.0.1/musl-cross-make/build/local/x86_64-linux-musl/obj_binutils/binutils/objdump RANLIB_FOR_TARGET=/home/xlings/.xlings_data/xim/xpkgs/musl-cross-make/0.0.1/musl-cross-make/build/local/x86_64-linux-musl/obj_binutils/binutils/ranlib READELF_FOR_TARGET=/home/xlings/.xlings_data/xim/xpkgs/musl-cross-make/0.0.1/musl-cross-make/build/local/x86_64-linux-musl/obj_binutils/binutils/readelf STRIP_FOR_TARGET=/home/xlings/.xlings_data/xim/xpkgs/musl-cross-make/0.0.1/musl-cross-make/build/local/x86_64-linux-musl/obj_binutils/binutils/strip-new --build=x86_64-pc-linux-muslxx --host=x86_64-pc-linux-muslxx
Thread model: posix
Supported LTO compression algorithms: zlib
gcc version 15.1.0 (GCC)
COMPILER_PATH=/home/xlings/.xlings_data/xim/xpkgs/musl-gcc/15.1.0/bin/../libexec/gcc/x86_64-linux-musl/15.1.0/:/home/xlings/.xlings_data/xim/xpkgs/musl-gcc/15.1.0/bin/../libexec/gcc/:/home/xlings/.xlings_data/xim/xpkgs/musl-gcc/15.1.0/bin/../lib/gcc/x86_64-linux-musl/15.1.0/../../../../x86_64-linux-musl/bin/
LIBRARY_PATH=/home/xlings/.xlings_data/xim/xpkgs/musl-gcc/15.1.0/bin/../lib/gcc/x86_64-linux-musl/15.1.0/:/home/xlings/.xlings_data/xim/xpkgs/musl-gcc/15.1.0/bin/../lib/gcc/:/home/xlings/.xlings_data/xim/xpkgs/musl-gcc/15.1.0/bin/../lib/gcc/x86_64-linux-musl/15.1.0/../../../../x86_64-linux-musl/lib/:/home/xlings/.xlings_data/xim/xpkgs/musl-gcc/15.1.0/bin/../x86_64-linux-musl/lib/
COLLECT_GCC_OPTIONS='-v' '-mtune=generic' '-march=x86-64' '-dumpdir' 'a.'
 /home/xlings/.xlings_data/xim/xpkgs/musl-gcc/15.1.0/bin/../libexec/gcc/x86_64-linux-musl/15.1.0/collect2 --sysroot=/home/xlings/.xlings_data/xim/xpkgs/musl-gcc/15.1.0/bin/../x86_64-linux-musl --eh-frame-hdr -m elf_x86_64 -dynamic-linker /home/xlings/.xlings_data/lib/ld-musl-x86_64.so.1 /home/xlings/.xlings_data/xim/xpkgs/musl-gcc/15.1.0/bin/../lib/gcc/x86_64-linux-musl/15.1.0/../../../../x86_64-linux-musl/lib/crt1.o /home/xlings/.xlings_data/xim/xpkgs/musl-gcc/15.1.0/bin/../lib/gcc/x86_64-linux-musl/15.1.0/../../../../x86_64-linux-musl/lib/crti.o /home/xlings/.xlings_data/xim/xpkgs/musl-gcc/15.1.0/bin/../lib/gcc/x86_64-linux-musl/15.1.0/crtbegin.o -L/home/xlings/.xlings_data/xim/xpkgs/musl-gcc/15.1.0/bin/../lib/gcc/x86_64-linux-musl/15.1.0 -L/home/xlings/.xlings_data/xim/xpkgs/musl-gcc/15.1.0/bin/../lib/gcc -L/home/xlings/.xlings_data/xim/xpkgs/musl-gcc/15.1.0/bin/../lib/gcc/x86_64-linux-musl/15.1.0/../../../../x86_64-linux-musl/lib -L/home/xlings/.xlings_data/xim/xpkgs/musl-gcc/15.1.0/bin/../x86_64-linux-musl/lib --enable-new-dtags -rpath /home/xlings/.xlings_data/lib -lgcc --push-state --as-needed -lgcc_s --pop-state -lc -lgcc --push-state --as-needed -lgcc_s --pop-state /home/xlings/.xlings_data/xim/xpkgs/musl-gcc/15.1.0/bin/../lib/gcc/x86_64-linux-musl/15.1.0/crtend.o /home/xlings/.xlings_data/xim/xpkgs/musl-gcc/15.1.0/bin/../lib/gcc/x86_64-linux-musl/15.1.0/../../../../x86_64-linux-musl/lib/crtn.o
/home/xlings/.xlings_data/xim/xpkgs/musl-gcc/15.1.0/bin/../lib/gcc/x86_64-linux-musl/15.1.0/../../../../x86_64-linux-musl/bin/ld: /home/xlings/.xlings_data/xim/xpkgs/musl-gcc/15.1.0/bin/../lib/gcc/x86_64-linux-musl/15.1.0/../../../../x86_64-linux-musl/lib/crt1.o: in function `_start_c':
crt1.c:(.text._start_c+0x15): undefined reference to `main'
collect2: error: ld returned 1 exit status
[root@localhost 15.1.0]#  gcc --version
x86_64-linux-musl-gcc (GCC) 15.1.0
Copyright (C) 2025 Free Software Foundation, Inc.
This is free software; see the source for copying conditions.  There is NO
warranty; not even for MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.

[root@localhost 15.1.0]# g++ --version
x86_64-linux-musl-g++ (GCC) 15.1.0
Copyright (C) 2025 Free Software Foundation, Inc.
This is free software; see the source for copying conditions.  There is NO
warranty; not even for MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.

```

## 1.5 Check gcc version
```dotnetcli
gcc --version
```
## 1.6 change source
```
xlings self config --res-server https://gitcode.com/xlings-res
xlings self config --res-server https://github.com/xlings-res
```
## 1.7  official website

The software is still in the research and development stage, so there may be some minor issues.

https://github.com/d2learn/xlings

enjoy it !
