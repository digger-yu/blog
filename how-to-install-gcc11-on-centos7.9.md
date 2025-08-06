

devtoolset的方式安装，操作简单，官方支持，缺点是需要手动添加环境变量
```
[root@localhost ~]# cat /etc/centos-release
CentOS Linux release 7.9.2009 (Core)
```
# 1. 添加repo
https://mirrors.aliyun.com/centos/7/sclo/x86_64/

```
cat > /etc/yum.repos.d/CentOS-SCLo-scl.repo << EOF
[centos-sclo-sclo]
name=CentOS-7 - SCLo sclo
baseurl=https://mirrors.aliyun.com/centos/7/sclo/x86_64/sclo/
# mirrorlist=https://mirrors.aliyun.com?arch=$basearch&release=7&repo=sclo-sclo
gpgcheck=0
enabled=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-SIG-SCLo
EOF
```
## 1.2 添加repo
https://mirrors.aliyun.com/centos/7/sclo/x86_64/rh/Packages/d
```
cat > /etc/yum.repos.d/CentOS-SCLo-scl-rh.repo << EOF
[centos-sclo-rh]
name=CentOS-7 - SCLo rh
baseurl=https://mirrors.aliyun.com/centos/7/sclo/x86_64/rh/
#mirrorlist=https://mirrors.aliyun.com?arch=$basearch&release=7&repo=sclo-rh
gpgcheck=0
enabled=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-SIG-SCLo
EOF
```
# 2. 安装

```
yum clean all && yum makecache
yum list devtoolset-*-gcc
yum -y install centos-release-scl
yum -y install devtoolset-11-gcc devtoolset-11-gcc-c++ devtoolset-11-binutils
```

# 3. 配置环境变量
```
临时生效
scl enable devtoolset-11 bash

永久生效
echo "source /opt/rh/devtoolset-11/enable" >>/etc/profile
source ~/.bashrc
```

# 4. 验证
```
[root@localhost ~]# gcc -v
Using built-in specs.
COLLECT_GCC=gcc
COLLECT_LTO_WRAPPER=/opt/rh/devtoolset-11/root/usr/libexec/gcc/x86_64-redhat-linux/11/lto-wrapper
Target: x86_64-redhat-linux
Configured with: ../configure --enable-bootstrap --enable-languages=c,c++,fortran,lto --prefix=/opt/rh/devtoolset-11/root/usr --mandir=/opt/rh/devtoolset-11/root/usr/share/man --infodir=/opt/rh/devtoolset-11/root/usr/share/info --with-bugurl=http://bugzilla.redhat.com/bugzilla --enable-shared --enable-threads=posix --enable-checking=release --enable-multilib --with-system-zlib --enable-__cxa_atexit --disable-libunwind-exceptions --enable-gnu-unique-object --enable-linker-build-id --with-gcc-major-version-only --with-linker-hash-style=gnu --with-default-libstdcxx-abi=gcc4-compatible --enable-plugin --enable-initfini-array --with-isl=/builddir/build/BUILD/gcc-11.2.1-20220127/obj-x86_64-redhat-linux/isl-install --enable-gnu-indirect-function --with-tune=generic --with-arch_32=x86-64 --build=x86_64-redhat-linux
Thread model: posix
Supported LTO compression algorithms: zlib
gcc version 11.2.1 20220127 (Red Hat 11.2.1-9) (GCC) 
[root@localhost ~]# g++ -v 
Using built-in specs.
COLLECT_GCC=g++
COLLECT_LTO_WRAPPER=/opt/rh/devtoolset-11/root/usr/libexec/gcc/x86_64-redhat-linux/11/lto-wrapper
Target: x86_64-redhat-linux
Configured with: ../configure --enable-bootstrap --enable-languages=c,c++,fortran,lto --prefix=/opt/rh/devtoolset-11/root/usr --mandir=/opt/rh/devtoolset-11/root/usr/share/man --infodir=/opt/rh/devtoolset-11/root/usr/share/info --with-bugurl=http://bugzilla.redhat.com/bugzilla --enable-shared --enable-threads=posix --enable-checking=release --enable-multilib --with-system-zlib --enable-__cxa_atexit --disable-libunwind-exceptions --enable-gnu-unique-object --enable-linker-build-id --with-gcc-major-version-only --with-linker-hash-style=gnu --with-default-libstdcxx-abi=gcc4-compatible --enable-plugin --enable-initfini-array --with-isl=/builddir/build/BUILD/gcc-11.2.1-20220127/obj-x86_64-redhat-linux/isl-install --enable-gnu-indirect-function --with-tune=generic --with-arch_32=x86-64 --build=x86_64-redhat-linux
Thread model: posix
Supported LTO compression algorithms: zlib
gcc version 11.2.1 20220127 (Red Hat 11.2.1-9) (GCC) 
```
