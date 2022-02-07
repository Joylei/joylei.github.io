+++
title = "WSL ubuntu build padavan"
date=2020-05-10

[taxonomies]
categories=["Windows"]
tags=["WSL", "ubuntu", "padavan", "router"]
+++

首先说结论，折腾了半天，报了一堆错，放弃了。建议直接完整ubuntu环境，很顺利编译。

## git repository

```
https://gitlab.com/dm38/padavan-ng.git
```

## 编译

首先遇到的问题是不能 build toolchain。
运行`./clean_sources.sh`，报错：

```
=================REMOVE-OLD-BUILD-TREE==================
make: /usr/bin/bash: Command not found
make: /usr/bin/bash: Command not found
/mnt/d/padavan-ng/toolchain/scripts/scripts.mk:63: recipe for target 'distclean' failed
make: *** [distclean] Error 127

```

运行`./build_toolchain.sh`报错：

```
================ START BUILDING TOOLCHAIN ==============
============== CLEANING OLD BUILD TREE =================
make: /usr/bin/bash: Command not found
make: /usr/bin/bash: Command not found
ct-ng:333: recipe for target 'clean' failed
make: *** [clean] Error 127
make: /usr/bin/bash: Command not found
make: *** No rule to make target 'mipsel-linux-uclibc'.  Stop.

```

查看toolchain目录下用到的是crosstool-ng。rosstool-ng文档说明需要执行：

- ./bootstrap
- ./configure --enable-local
- make

但是最后make时报错：

```
CDPATH="${ZSH_VERSION+.}:" && cd . && /bin/bash /opt/rt-n56u/toolchain/scripts/missing aclocal-1.16
/opt/rt-n56u/toolchain/scripts/missing: line 81: aclocal-1.16: command not found
WARNING: 'aclocal-1.16' is missing on your system.
         You should only need it if you modified 'acinclude.m4' or
         'configure.ac' or m4 files included by 'configure.ac'.
         The 'aclocal' program is part of the GNU Automake package:
         <http://www.gnu.org/software/automake>
         It also requires GNU Autoconf, GNU m4 and Perl in order to run:
         <http://www.gnu.org/software/autoconf>
         <http://www.gnu.org/software/m4/>
         <http://www.perl.org/>
Makefile:2599: recipe for target 'aclocal.m4' failed
make: *** [aclocal.m4] Error 127
```

搜索后发现需要执行命令：`autoreconf -r -i`

```
Copying file ABOUT-NLS
Copying file scripts/config.rpath
Copying file m4/codeset.m4
Copying file m4/extern-inline.m4
Copying file m4/fcntl-o.m4
Copying file m4/glibc2.m4
Copying file m4/glibc21.m4
Copying file m4/intdiv0.m4
Copying file m4/intl.m4
Copying file m4/intldir.m4
Copying file m4/intmax.m4
Copying file m4/inttypes-pri.m4
Copying file m4/inttypes_h.m4
Copying file m4/lcmessage.m4
Copying file m4/lock.m4
Copying file m4/longlong.m4
Copying file m4/printf-posix.m4
Copying file m4/size_max.m4
Copying file m4/stdint_h.m4
Copying file m4/threadlib.m4
Copying file m4/uintmax_t.m4
Copying file m4/visibility.m4
Copying file m4/wchar_t.m4
Copying file m4/wint_t.m4
Copying file m4/xsize.m4
Creating directory po
Copying file po/Makefile.in.in
Copying file po/Makevars.template
Copying file po/Rules-quot
Copying file po/boldquot.sed
Copying file po/en@boldquot.header
Copying file po/en@quot.header
Copying file po/insert-header.sin
Copying file po/quot.sed
Copying file po/remove-potcdate.sin
automake: warnings are treated as errors
Makefile.am:6: warning: AM_GNU_GETTEXT used but 'po' not in SUBDIRS
autoreconf: automake failed with exit status: 1
```

再次执行make

```
 cd . && /bin/bash /opt/rt-n56u/toolchain/scripts/missing automake-1.16 --foreign
/opt/rt-n56u/toolchain/scripts/missing: line 81: automake-1.16: command not found
WARNING: 'automake-1.16' is missing on your system.
         You should only need it if you modified 'Makefile.am' or
         'configure.ac' or m4 files included by 'configure.ac'.
         The 'automake' program is part of the GNU Automake package:
         <http://www.gnu.org/software/automake>
         It also requires GNU Autoconf, GNU m4 and Perl in order to run:
         <http://www.gnu.org/software/autoconf>
         <http://www.gnu.org/software/m4/>
         <http://www.perl.org/>
Makefile:2570: recipe for target 'Makefile.in' failed
make: *** [Makefile.in] Error 1

```

看起来automake版本不对，确认版本：`automake --version`

```
automake (GNU automake) 1.15.1
```

`apt search automake`发现源里最高版本是1.15

下载安装automake

```sh
sudo apt-get autoremove automake
wget http://ftp.gnu.org/gnu/automake/automake-1.16.2.tar.gz
tar -xf automake-1.16.2.tar.gz
cd automake-1.16.2
./bootstrap
./configure
make
sudo make install
```
