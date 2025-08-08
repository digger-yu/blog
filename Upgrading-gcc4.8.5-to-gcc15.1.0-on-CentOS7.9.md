# Upgrade gcc4.8.5 to gcc 15.1.0 on CentOS 7.9 using the source code package. 

本文记录了在centos7.9上升级gcc4.8.5到gcc15.1.0的过程。供参考
This article documents the process of upgrading gcc 4.8.5 to gcc 15.1.0 using source packages on CentOS 7.9. For reference.

# 1.1 下载源码包
```bash
官方站点 https://gcc.gnu.org/pub/gcc/releases/gcc-15.1.0/gcc-15.1.0.tar.gz
wget https://mirrors.aliyun.com/gnu/gcc/gcc-15.1.0/gcc-15.1.0.tar.gz
tar -zxvf gcc-15.1.0.tar.gz && cd gcc-15.1.0
```

# 1.2 安装依赖
```bash
yum groupinstall "Development Tools" -y
yum install -y make tar gawk flex bison perl python3 dejagnu autoconf automake texinfo sphinx gettext-devel bzip2-devel zlib-devel kernel-headers gmp-devel mpfr-devel libmpc-devel isl-devel gcc gcc-c++ wget xz libgcc glibc-devel glibc-headers libstdc++-devel
```
详细参考 https://gcc.gnu.org/install/prerequisites.html

# 1.3 依赖关系
| 软件包名称          | 版本要求          | 备注                                                                 |
|---------------------|-------------------|----------------------------------------------------------------------|
| GCC                 | 5.4+（引导）      | 构建当前GCC需C++14支持；旧版本需更低C++标准（如GCC 15前需C++11）     |
| GNAT                | 5.1+              | 构建Ada编译器必需（GCC内置）                                         |
| GDC                 | 9.4+              | 构建D编译器必需（GCC内置）                                           |
| Python              | -                 | 配置RISC-V非规范架构时需要                                           |
| Python3             | -                 | 生成GM2完整文档时必需                                                |
| GNU flex            | ≥2.5.4            | 构建词法分析模块                                                     |
| GNU awk             | 最新版本          | POSIX/SVR4 awk，建议使用GNU awk                                      |
| GNU binutils        | ≥2.35             | 启用LTO时必需                                                        |
| gzip/bzip2          | ≥1.2.4 / ≥1.0.2   | 解压GCC源码                                                          |
| GNU make            | ≥3.80             | 构建必需                                                             |
| GNU tar             | ≥1.14             | 部分平台解压源码必需                                                 |
| Perl                | ≥5.6.1            | 特定场景（如Darwin/Solaris）                                         |
| zstd                | -                 | 启用LTO压缩时必需                                                    |
| GMP                 | ≥4.3.2            | 数学运算库（必须）                                                   |
| MPFR                | ≥3.1.0            | 浮点运算库（必须）                                                   |
| MPC                 | ≥1.0.1            | 复数运算库（必须）                                                   |
| isl                 | ≥0.15             | Graphite循环优化必需                                                 |
| gettext             | ≥0.22             | 国际化支持（必须）                                                   |
| autoconf            | ≥2.69             | 修改GCC源码时必需                                                    |
| automake            | ≥1.15.1           | 修改GCC Makefile时必需                                               |
| gperf               | ≥2.7.2            | 生成哈希函数时必需                                                   |
| DejaGnu             | ≥1.5.3            | 测试框架必需                                                         |
| expect              | -                 | 运行测试套件必需                                                     |
| tcl                 | -                 | 运行测试套件必需                                                     |
| Texinfo             | ≥4.7              | 文档生成工具（必须）                                                 |
| TeX                 | -                 | PDF文档生成必需                                                      |
| Sphinx              | ≥1.0              | JIT文档生成必需                                                      |

### 条件性依赖说明：
1. **Python/GNU awk**：仅在特定配置（如非规范RISC-V架构）时需要  
2. **gettext**：启用`--enable-nls`国际化时必需  
3. **测试工具链（expect/tcl）**：仅用于GCC官方测试套件  
4. **文档工具（Sphinx/Texinfo）**：仅用于生成开发文档  

### 注意事项：
- 所有`≥`符号表示"等于或高于指定版本"
- 加粗包名为核心依赖，其余为可选或场景依赖
- 建议优先使用系统包管理器安装（如`apt-get/yum install g++-multilib`）

# 1.4 安装gcc依赖库
```bash
 使用gcc里面的下载脚本
./contrib/download_prerequisites
[root@localhost gcc-15.1.0]# ./contrib/download_prerequisites
2025-08-07 07:04:51 URL:https://gcc.gnu.org/pub/gcc/infrastructure/gettext-0.22.tar.gz [26105696/26105696] -> "gettext-0.22.tar.gz" [1]
2025-08-07 07:04:55 URL:https://gcc.gnu.org/pub/gcc/infrastructure/gmp-6.2.1.tar.bz2 [2493916/2493916] -> "gmp-6.2.1.tar.bz2" [1]
2025-08-07 07:04:58 URL:https://gcc.gnu.org/pub/gcc/infrastructure/mpfr-4.1.0.tar.bz2 [1747243/1747243] -> "mpfr-4.1.0.tar.bz2" [1]
2025-08-07 07:05:01 URL:https://gcc.gnu.org/pub/gcc/infrastructure/mpc-1.2.1.tar.gz [838731/838731] -> "mpc-1.2.1.tar.gz" [1]
2025-08-07 07:05:05 URL:https://gcc.gnu.org/pub/gcc/infrastructure/isl-0.24.tar.bz2 [2261594/2261594] -> "isl-0.24.tar.bz2" [1]
gettext-0.22.tar.gz: 确定
gmp-6.2.1.tar.bz2: 确定
mpfr-4.1.0.tar.bz2: 确定
mpc-1.2.1.tar.gz: 确定
isl-0.24.tar.bz2: 确定
All prerequisites downloaded successfully.

脚本其实就是下载并解压了这些软件包，并且创建了软连接
-rw-r--r--  1 root root    26105696 6月  17 2023 gettext-0.22.tar.gz
drwxrwxr-x 10 root root        4096 6月  17 2023 gettext-0.22
-rw-r--r--  1 root root     2261594 5月   2 2021 isl-0.24.tar.bz2
drwxrwxr-x 11 1000    1000    12288 5月   2 2021 isl-0.24
-rw-r--r--  1 root root     2493916 11月 15 2020 gmp-6.2.1.tar.bz2
drwxrwxr-x 15 1006    1006     4096 11月 15 2020 gmp-6.2.1
-rw-r--r--  1 root root      838731 10月 23 2020 mpc-1.2.1.tar.gz
drwxr-xr-x  8 1001 polkitd      319 10月 21 2020 mpc-1.2.1
-rw-r--r--  1 root root     1747243 7月  10 2020 mpfr-4.1.0.tar.bz2
drwxr-xr-x  9 1000    1000     4096 7月  10 2020 mpfr-4.1.0
lrwxrwxrwx  1 root root          11 8月   7 07:05 isl -> ./isl-0.24/
lrwxrwxrwx  1 root root          12 8月   7 07:05 mpc -> ./mpc-1.2.1/
lrwxrwxrwx  1 root root          13 8月   7 07:05 mpfr -> ./mpfr-4.1.0/
lrwxrwxrwx  1 root root          12 8月   7 07:05 gmp -> ./gmp-6.2.1/
lrwxrwxrwx  1 root root          15 8月   7 07:05 gettext -> ./gettext-0.22/
```
# 1.5 配置编译选项
```bash
./configure --prefix=/usr/local/gcc-15.1.0 --enable-checking=release --enable-languages=c,c++ --disable-multilib
```

# 1.5.1 GCC15.1.0配置参数说明
#### 一、基础配置参数
| 参数                  | 说明                                       |
|-----------------------|--------------------------------------------|
| `--prefix=dirname`    | 指定顶级安装目录（默认 `/usr/local` ）      |
| `--exec-prefix=dirname` | 指定架构相关文件的安装目录（默认同 `prefix` ）|
| `--bindir=dirname`    | 安装用户可执行文件的目录（默认 `exec-prefix/bin` ）|
| `--libdir=dirname`    | 安装目标代码库的目录（默认 `exec-prefix/lib` ）|

#### 二、原生编译配置
| 参数                  | 说明                                       |
|-----------------------|--------------------------------------------|
| `--enable-bootstrap`  | 启用 3 阶段引导构建（默认开启）             |
| `--disable-bootstrap` | 禁用引导构建（需显式指定）                  |
| `--with-gcc-major-version-only` | 仅使用 GCC 主版本号作为路径标识              |
| `--with-native-system-header-dir=dirname` | 指定独立系统头文件目录（隔离系统依赖）       |

#### 三、交叉编译配置
| 参数                  | 说明                                       |
|-----------------------|--------------------------------------------|
| `--host=ARCH`         | 指定构建主机架构（如 `x86_64-linux-gnu` ）  |
| `--target=ARCH`       | 指定目标架构（如 `arm-linux-gnueabihf` ）   |
| `--with-sysroot=PATH` | 设置目标系统根目录路径                      |
| `--with-headers=PATH` | 指定目标系统头文件路径                      |
| `--with-libs=PATH`    | 指定目标系统库文件路径                      |

#### 四、优化与调试参数
| 参数                  | 说明                                       |
|-----------------------|--------------------------------------------|
| `-O0`                 | 禁用优化（调试模式）                        |
| `-O2`                 | 默认优化级别（平衡速度与体积）              |
| `-O3`                 | 最高等级优化（可能增大体积）                |
| `-g`                  | 生成调试信息                                |
| `-fsanitize=address`  | 启用地址消毒剂（检测内存错误）              |
| `-fsanitize=undefined`| 启用未定义行为消毒剂（检测未定义操作）      |

#### 五、特殊构建模式
| 参数                  | 说明                                       |
|-----------------------|--------------------------------------------|
| `--enable-shared`     | 构建共享库版本                              |
| `--enable-static`     | 优先构建静态库版本                          |
| `--disable-multilib`  | 禁用多架构支持（简化构建）                  |
| `--with-multilib-list=list` | 指定多架构库列表（如 `ilp32,lp64` ）        |

#### 六、线程与安全配置
| 参数                  | 说明                                       |
|-----------------------|--------------------------------------------|
| `--enable-threads=lib` | 选择线程库（`posix`/`win32`/`single`）      |
| `--enable-tls`        | 启用线程局部存储（TLS）支持                 |
| `--enable-pie`        | 构建位置无关可执行文件（增强 ASLR 防护）    |

#### 七、高级功能参数
| 参数                  | 说明                                       |
|-----------------------|--------------------------------------------|
| `--with-gmp=PATH`     | 指定 GMP 库路径（依赖数学运算时需要）       |
| `--with-isl=PATH`     | 指定 ISL 库路径（用于优化算法）             |
| `--enable-lto`        | 启用链接时优化（LTO）                       |
| `--enable-cet`        | 启用控制流保护（Intel CET）                 |

#### 八、环境变量影响
| 变量                  | 说明                                       |
|-----------------------|--------------------------------------------|
| `CC`                  | 指定宿主编译器（可能干扰跨平台构建）        |
| `CFLAGS`              | 传递额外编译参数（如 `-march=native` ）     |
| `LDFLAGS`             | 传递额外链接参数（如 `-L/usr/local/lib` ）  |

#### 九、目标平台适配
| 参数                  | 说明                                       |
|-----------------------|--------------------------------------------|
| `--with-arch=CPU`     | 指定默认目标 CPU（如 `armv8-a` ）           |
| `--with-fpu=type`     | 启用浮点单元支持（如 `vfpv4` ）             |
| `--with-endian=mode`  | 指定字节序（`big`/`little`/`bi-endian`）    |

#### 十、开发者专用选项
| 参数                  | 说明                                       |
|-----------------------|--------------------------------------------|
| `--enable-maintainer-mode` | 启用维护者模式（重新生成自动生成文件）      |
| `--enable-checking=level` | 启用内部一致性检查（`yes`/`release`/`all`） |
| `--with-diagnostics-color=choice` | 控制诊断信息颜色输出（`auto`/`always`）     |

> **注意事项**：
> 1. 参数优先级：命令行参数 > 环境变量 > 配置缓存
> 2. 跨平台构建建议：先构建宿主 GCC，再使用宿主 GCC 构建目标 GCC
> 3. 内存敏感场景：可添加 `MAKEFLAGS="-j1"` 限制并行度
> 4. 多架构支持：使用 `--enable-multilib` 并指定 `--with-multilib-list`
> 5. 安全加固：推荐启用 `--enable-pie` 和 `--enable-cet`
# 1.5.2 GCC 15.1.0编译参数说明


#### 一、基础配置参数
| 参数                  | 说明                                       |
|-----------------------|--------------------------------------------|
| `--disable-bootstrap` | 禁用引导构建（默认启用3阶段自举）          |
| `--enable-bootstrap`  | 强制启用引导构建（交叉编译时需指定）         |
| `--with-build-config=NAME` | 应用预定义构建配置（如`bootstrap-O1`）       |

#### 二、原生编译器参数
| 参数                  | 说明                                       |
|-----------------------|--------------------------------------------|
| `BOOT_CFLAGS='-O'`    | 阶段2/3编译时添加优化标志（节省40%空间）     |
| `CFLAGS_FOR_TARGET`   | 设置目标库编译标志（不影响引导阶段）         |
| `STAGE1_TFLAGS`       | 修复阶段1编译器问题时覆盖临时标志            |
| `--enable-languages=C,C++` | 仅构建指定语言的编译器（默认全开）           |

#### 三、交叉编译参数
| 参数                  | 说明                                       |
|-----------------------|--------------------------------------------|
| `--host=ARCH`         | 指定构建主机架构（如x86_64-linux-gnu）       |
| `--target=ARCH`       | 指定目标架构（如arm-linux-gnueabihf）        |
| `--with-sysroot=PATH` | 设置目标系统根目录路径                       |
| `--with-headers=PATH` | 指定目标系统头文件路径                       |
| `--with-libs=PATH`    | 指定目标系统库文件路径                       |

#### 四、优化与调试参数
| 参数                  | 说明                                       |
|-----------------------|--------------------------------------------|
| `-O0`                 | 禁用优化（调试模式）                        |
| `-O2`                 | 默认优化级别（平衡速度与体积）               |
| `-O3`                 | 最高等级优化（可能增大体积）                |
| `-g`                  | 生成调试信息                                |
| `-fsanitize=address`  | 启用地址消毒剂（检测内存错误）               |
| `-fsanitize=undefined`| 启用未定义行为消毒剂（检测未定义操作）       |

#### 五、特殊构建模式
| 参数                  | 说明                                       |
|-----------------------|--------------------------------------------|
| `make profiledbootstrap` | 使用性能分析数据进行引导优化               |
| `make bootstrap-debug`   | 对比阶段2/3编译结果（检测编译器自身错误）    |
| `make bootstrap-lto`     | 在引导过程中启用链接时优化（需链接器支持）   |
| `make bootstrap-lean`    | 减少引导构建生成的中间文件体积               |

#### 六、环境变量影响
| 变量                  | 说明                                       |
|-----------------------|--------------------------------------------|
| `CC`                  | 指定宿主编译器（可能干扰跨平台构建）         |
| `LDFLAGS`             | 传递额外链接参数（如`-L/usr/local/lib`）     |
| `CPPFLAGS`            | 预处理器参数（如`-I/usr/local/include`）     |

#### 七、高级功能参数
| 参数                  | 说明                                       |
|-----------------------|--------------------------------------------|
| `--enable-shared`     | 构建共享库版本                             |
| `--enable-static`     | 优先构建静态库版本                         |
| `--disable-multilib`  | 禁用多架构支持（简化构建过程）               |
| `--with-gmp=PATH`     | 指定GMP库路径（依赖数学运算时需要）          |

> **注意**：  
> 1. 参数优先级：命令行参数 > 环境变量 > 配置缓存  
> 2. 跨平台构建建议：先构建宿主GCC，再使用宿主GCC构建目标GCC  
> 3. 内存敏感场景：可添加 `MAKEFLAGS="-j1"` 限制并行度  
> 4. 编译时间长：建议在多核CPU上并行编译（如`-j4`）  
> 5. 磁盘空间要求：建议至少20GB可用空间  
> 6. 编译日志：建议使用`make V=1 2>&1 | tee make.log`记录详细编译信息  
> 7. 编译错误：如果遇到错误，根据错误信息定位问题并解决  
> 8. 编译成功：如果没有错误，恭喜你，GCC编译成功！  

# 1.6 编译安装gcc 15.1.0
```bash
make -j4 && make install
make V=1 2>&1 | tee make.log 
```

# 1.7 有个报错
```bash
configure: error: uint64_t or int64_t not found
make[2]: *** [configure-stage1-gcc] 错误 1
make[2]: 离开目录“/root/gcc-15.1.0/build”
make[1]: *** [stage1-bubble] 错误 2
make[1]: 离开目录“/root/gcc-15.1.0/build”
make: *** [all] 错误 2
```
# 1.7.1 报错解决方法
```bash
ISO C++14 compiler
Necessary to bootstrap GCC. GCC 5.4 or newer has sufficient support for used C++14 features.
Versions of GCC prior to 15 allow bootstrapping with an ISO C++11 compiler, versions prior to 10.5 allow bootstrapping with an ISO C++98 compiler, and versions prior to 4.8 allow bootstrapping with an ISO C89 compiler.
If you need to build an intermediate version of GCC in order to bootstrap current GCC, consider GCC 9.5: it can build the current Ada and D compilers, and was also the version that declared C++17 support stable.
To build all languages in a cross-compiler or other configuration where 3-stage bootstrap is not performed, you need to start with an existing GCC binary (of a new enough version) because source code for language frontends other than C might use GCC extensions.
```
# 1.7.2 准备升级gcc5.5.0
```bash
#yum install glibc-static libstdc++-static binutils
cd 
wget https://ftp.gnu.org/gnu/gcc/gcc-5.5.0/gcc-5.5.0.tar.gz
47  tar -zxvf gcc-5.5.0.tar.gz 
48  cd gcc-5.5.0/
49  ./contrib/download_prerequisites 
50  mkdir build && cd build 
51  ../configure --prefix=/usr/local/gcc-5.5.0 --disable-multilib --enable-languages=c,c++
52  make -j4 && make install 

```
# 1.7.3 install gcc5.5.0完成
```bash

----------------------------------------------------------------------
Libraries have been installed in:
   /usr/local/gcc-5.5.0/lib/../lib64

If you ever happen to want to link against installed libraries
in a given directory, LIBDIR, you must either use libtool, and
specify the full pathname of the library, or use the `-LLIBDIR'
flag during linking and do at least one of the following:
   - add LIBDIR to the `LD_LIBRARY_PATH' environment variable
     during execution
   - add LIBDIR to the `LD_RUN_PATH' environment variable
     during linking
   - use the `-Wl,-rpath -Wl,LIBDIR' linker flag
   - have your system administrator add LIBDIR to `/etc/ld.so.conf'

See any operating system documentation about shared libraries for
more information, such as the ld(1) and ld.so(8) manual pages.
----------------------------------------------------------------------
make[4]: 对“install-data-am”无需做任何事。
make[4]: 离开目录“/root/gcc-5.5.0/build/x86_64-unknown-linux-gnu/libatomic”
make[3]: 离开目录“/root/gcc-5.5.0/build/x86_64-unknown-linux-gnu/libatomic”
make[2]: 离开目录“/root/gcc-5.5.0/build/x86_64-unknown-linux-gnu/libatomic”
make[1]: 离开目录“/root/gcc-5.5.0/build”
[root@localhost build]# gcc -v
使用内建 specs。
COLLECT_GCC=gcc
COLLECT_LTO_WRAPPER=/usr/libexec/gcc/x86_64-redhat-linux/4.8.5/lto-wrapper
目标：x86_64-redhat-linux
配置为：../configure --prefix=/usr --mandir=/usr/share/man --infodir=/usr/share/info --with-bugurl=http://bugzilla.redhat.com/bugzilla --enable-bootstrap --enable-shared --enable-threads=posix --enable-checking=release --with-system-zlib --enable-__cxa_atexit --disable-libunwind-exceptions --enable-gnu-unique-object --enable-linker-build-id --with-linker-hash-style=gnu --enable-languages=c,c++,objc,obj-c++,java,fortran,ada,go,lto --enable-plugin --enable-initfini-array --disable-libgcj --with-isl=/builddir/build/BUILD/gcc-4.8.5-20150702/obj-x86_64-redhat-linux/isl-install --with-cloog=/builddir/build/BUILD/gcc-4.8.5-20150702/obj-x86_64-redhat-linux/cloog-install --enable-gnu-indirect-function --with-tune=generic --with-arch_32=x86-64 --build=x86_64-redhat-linux
线程模型：posix
gcc 版本 4.8.5 20150623 (Red Hat 4.8.5-44) (GCC) 
[root@localhost build]# 
```
# 1.7.4 更新gcc5.5.0到环境变量
```bash
54  echo 'export PATH=/usr/local/gcc-5.5.0/bin:$PATH' >> ~/.bashrc
55  echo 'export LD_LIBRARY_PATH=/usr/local/gcc-5.5.0/lib64:$LD_LIBRARY_PATH' >> ~/.bashrc
56  source ~/.bashrc
```

# 1.7.5  验证版本
```bash
[root@localhost build]# gcc -v
使用内建 specs。
COLLECT_GCC=gcc
COLLECT_LTO_WRAPPER=/usr/local/gcc-5.5.0/libexec/gcc/x86_64-unknown-linux-gnu/5.5.0/lto-wrapper
目标：x86_64-unknown-linux-gnu
配置为：../configure --prefix=/usr/local/gcc-5.5.0 --disable-multilib --enable-languages=c,c++
线程模型：posix
gcc 版本 5.5.0 (GCC) 
[root@localhost build]# 
```

 升级gcc5.5.0后验证
# 1.8 继续升级到gcc15.1.0
  ```bash
    cd /root/gcc-15.1.0/
    make clean
    #make distclean
    #../configure --prefix=/usr/local/gcc-15.1.0 --enable-checking=release --enable-languages=c,c++ --disable-multilib
    make -j4 && make install
    #make V=1 2>&1 | tee make.log 
  ```

```
libtool: install: /usr/bin/install -c .libs/libatomic.a /usr/local/gcc-15.1.0/lib/../lib64/libatomic.a
libtool: install: chmod 644 /usr/local/gcc-15.1.0/lib/../lib64/libatomic.a
libtool: install: ranlib  --plugin /root/gcc-15.1.0/build/./gcc/liblto_plugin.so /usr/local/gcc-15.1.0/lib/../lib64/libatomic.a
libtool: finish: PATH="/usr/local/gcc-5.5.0/bin:/usr/local/gcc-5.5.0/bin:/usr/local/gcc-5.5.0/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/root/bin:/sbin" ldconfig -n /usr/local/gcc-15.1.0/lib/../lib64
ldconfig: /usr/local/gcc-15.1.0/lib/../lib64/libstdc++.so.6.0.34-gdb.py is not an ELF file - it has the wrong magic bytes at the start.

----------------------------------------------------------------------
Libraries have been installed in:
   /usr/local/gcc-15.1.0/lib/../lib64

If you ever happen to want to link against installed libraries
in a given directory, LIBDIR, you must either use libtool, and
specify the full pathname of the library, or use the `-LLIBDIR'
flag during linking and do at least one of the following:
   - add LIBDIR to the `LD_LIBRARY_PATH' environment variable
     during execution
   - add LIBDIR to the `LD_RUN_PATH' environment variable
     during linking
   - use the `-Wl,-rpath -Wl,LIBDIR' linker flag
   - have your system administrator add LIBDIR to `/etc/ld.so.conf'

See any operating system documentation about shared libraries for
more information, such as the ld(1) and ld.so(8) manual pages.
----------------------------------------------------------------------
make[4]: 对“install-data-am”无需做任何事。
make[4]: 离开目录“/root/gcc-15.1.0/build/x86_64-pc-linux-gnu/libatomic”
make[3]: 离开目录“/root/gcc-15.1.0/build/x86_64-pc-linux-gnu/libatomic”
make[2]: 离开目录“/root/gcc-15.1.0/build/x86_64-pc-linux-gnu/libatomic”
make[1]: 离开目录“/root/gcc-15.1.0/build”
[root@localhost build]# 
```
# 1.9 配置环境变量
```bash
echo 'export PATH=/usr/local/gcc-15.1.0/bin:$PATH' >> ~/.bashrc
echo 'export LD_LIBRARY_PATH=/usr/local/gcc-15.1.0/lib64:$LD_LIBRARY_PATH' >> ~/.bashrc
source ~/.bashrc
```
# 2.0 查看gcc版本
```bash
gcc --version
g++ --version
```
```dotnetcli
[root@localhost build]# gcc -v 
使用内建 specs。
COLLECT_GCC=gcc
COLLECT_LTO_WRAPPER=/usr/local/gcc-15.1.0/libexec/gcc/x86_64-pc-linux-gnu/15.1.0/lto-wrapper
目标：x86_64-pc-linux-gnu
配置为：../configure --prefix=/usr/local/gcc-15.1.0 --enable-checking=release --enable-languages=c,c++ --disable-multilib
线程模型：posix
支持的 LTO 压缩算法：zlib
gcc 版本 15.1.0 (GCC) 
[root@localhost build]# g++ -v 
使用内建 specs。
COLLECT_GCC=g++
COLLECT_LTO_WRAPPER=/usr/local/gcc-15.1.0/libexec/gcc/x86_64-pc-linux-gnu/15.1.0/lto-wrapper
目标：x86_64-pc-linux-gnu
配置为：../configure --prefix=/usr/local/gcc-15.1.0 --enable-checking=release --enable-languages=c,c++ --disable-multilib
线程模型：posix
支持的 LTO 压缩算法：zlib
gcc 版本 15.1.0 (GCC) 
[root@localhost build]# 
```
