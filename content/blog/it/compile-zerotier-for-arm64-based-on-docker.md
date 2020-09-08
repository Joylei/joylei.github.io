+++
title="基于docker的arm64编译zerotier"
date=2020-06-12

[taxonomies]
categories=["Network"]
tags=["arm64","zerotier", "docker", "network", "build"]
+++
为了在路由器使用zerotier，但又不想安装entware，自己动手编译zerotier。路由器使用的梅林固件，cpu是arm64架构的。

## 注册qemu-user-static

```
❯ docker run --rm --privileged multiarch/qemu-user-static:register --reset
Setting /usr/bin/qemu-alpha-static as binfmt interpreter for alpha
Setting /usr/bin/qemu-arm-static as binfmt interpreter for arm
Setting /usr/bin/qemu-armeb-static as binfmt interpreter for armeb
Setting /usr/bin/qemu-sparc-static as binfmt interpreter for sparc
Setting /usr/bin/qemu-sparc32plus-static as binfmt interpreter for sparc32plus
Setting /usr/bin/qemu-sparc64-static as binfmt interpreter for sparc64
Setting /usr/bin/qemu-ppc-static as binfmt interpreter for ppc
Setting /usr/bin/qemu-ppc64-static as binfmt interpreter for ppc64
Setting /usr/bin/qemu-ppc64le-static as binfmt interpreter for ppc64le
Setting /usr/bin/qemu-m68k-static as binfmt interpreter for m68k
Setting /usr/bin/qemu-mips-static as binfmt interpreter for mips
Setting /usr/bin/qemu-mipsel-static as binfmt interpreter for mipsel
Setting /usr/bin/qemu-mipsn32-static as binfmt interpreter for mipsn32
Setting /usr/bin/qemu-mipsn32el-static as binfmt interpreter for mipsn32el
Setting /usr/bin/qemu-mips64-static as binfmt interpreter for mips64
Setting /usr/bin/qemu-mips64el-static as binfmt interpreter for mips64el
Setting /usr/bin/qemu-sh4-static as binfmt interpreter for sh4
Setting /usr/bin/qemu-sh4eb-static as binfmt interpreter for sh4eb
Setting /usr/bin/qemu-s390x-static as binfmt interpreter for s390x
Setting /usr/bin/qemu-aarch64-static as binfmt interpreter for aarch64
Setting /usr/bin/qemu-aarch64_be-static as binfmt interpreter for aarch64_be
Setting /usr/bin/qemu-hppa-static as binfmt interpreter for hppa
Setting /usr/bin/qemu-riscv32-static as binfmt interpreter for riscv32
Setting /usr/bin/qemu-riscv64-static as binfmt interpreter for riscv64
Setting /usr/bin/qemu-xtensa-static as binfmt interpreter for xtensa
Setting /usr/bin/qemu-xtensaeb-static as binfmt interpreter for xtensaeb
Setting /usr/bin/qemu-microblaze-static as binfmt interpreter for microblaze
Setting /usr/bin/qemu-microblazeel-static as binfmt interpreter for microblazeel
Setting /usr/bin/qemu-or1k-static as binfmt interpreter for or1k

```

## 准备ubuntu arm64源

默认的源更新太慢了，替换成阿里云的源。注意：这里对应的是ubuntu focal发行版；其它的发行版，请自行替换源地址。
创建文件`sources.list`，内容如下：

```
# See http://help.ubuntu.com/community/UpgradeNotes for how to upgrade to
# newer versions of the distribution.
deb http://mirrors.aliyun.com/ubuntu-ports/ focal main restricted
# deb-src http://archive.ubuntu.com/ubuntu/ focal main restricted

## Major bug fix updates produced after the final release of the
## distribution.
deb http://mirrors.aliyun.com/ubuntu-ports/ focal-updates main restricted
# deb-src http://archive.ubuntu.com/ubuntu/ focal-updates main restricted

## N.B. software from this repository is ENTIRELY UNSUPPORTED by the Ubuntu
## team. Also, please note that software in universe WILL NOT receive any
## review or updates from the Ubuntu security team.
deb http://mirrors.aliyun.com/ubuntu-ports/ focal universe
# deb-src http://archive.ubuntu.com/ubuntu/ focal universe
deb http://mirrors.aliyun.com/ubuntu-ports/ focal-updates universe
# deb-src http://archive.ubuntu.com/ubuntu/ focal-updates universe

## N.B. software from this repository is ENTIRELY UNSUPPORTED by the Ubuntu
## team, and may not be under a free licence. Please satisfy yourself as to
## your rights to use the software. Also, please note that software in
## multiverse WILL NOT receive any review or updates from the Ubuntu
## security team.
deb http://mirrors.aliyun.com/ubuntu-ports/ focal multiverse
# deb-src http://archive.ubuntu.com/ubuntu/ focal multiverse
deb http://mirrors.aliyun.com/ubuntu-ports/ focal-updates multiverse
# deb-src http://archive.ubuntu.com/ubuntu/ focal-updates multiverse

## N.B. software from this repository may not have been tested as
## extensively as that contained in the main release, although it includes
## newer versions of some applications which may provide useful features.
## Also, please note that software in backports WILL NOT receive any review
## or updates from the Ubuntu security team.
deb http://mirrors.aliyun.com/ubuntu-ports/ focal-backports main restricted universe multiverse
# deb-src http://archive.ubuntu.com/ubuntu/ focal-backports main restricted universe multiverse

## Uncomment the following two lines to add software from Canonical's
## 'partner' repository.
## This software is not part of Ubuntu, but is offered by Canonical and the
## respective vendors as a service to Ubuntu users.
# deb http://archive.canonical.com/ubuntu focal partner
# deb-src http://archive.canonical.com/ubuntu focal partner

deb http://mirrors.aliyun.com/ubuntu-ports/ focal-security main restricted
# deb-src http://security.ubuntu.com/ubuntu/ focal-security main restricted
deb http://mirrors.aliyun.com/ubuntu-ports/ focal-security universe
# deb-src http://security.ubuntu.com/ubuntu/ focal-security universe
deb http://mirrors.aliyun.com/ubuntu-ports/ focal-security multiverse
# deb-src http://security.ubuntu.com/ubuntu/ focal-security multiverse

```

## 准备Dockerfile
<https://hub.docker.com/u/multiarch/>

创建文件`Dockerfile`,内容如下：

```
FROM multiarch/ubuntu-core:arm64-focal

COPY sources.list /etc/apt/

RUN apt-get update -yqq \
  && apt-get install gcc g++ make curl git -y

VOLUME /tmp
WORKDIR /tmp

ENTRYPOINT ["tail", "-f", "/dev/null"]
```

可以自己修改Dockerfile，使用`apt-get`安装必要的包。

## 构建docker镜像

```
❯ docker build -t arm64:test .
```

## 查看镜像

```
❯ docker image ls
REPOSITORY                   TAG                 IMAGE ID            CREATED             SIZE
arm64                        test                1b6c48260def        3 minutes ago       287MB
<none>                       <none>              727fd2a54c5f        11 minutes ago      70.2MB
multiarch/ubuntu-core        arm64-focal         ec57294b3a5f        4 days ago          70.2MB
multiarch/qemu-user-static   register            a119103618d8        12 days ago         1.25MB
```

## 创建容器

```
❯ docker run -d --name arm64  -v /share/homes/admin/zerotier-dev/tmp:/tmp arm64:test
1123a7097cb7aa376a97ac3477b83dab95a44dd457e3daf5d21a82384209d66e
```

## 编译

### 在bash中执行命令

```
❯ docker exec -it arm64 /bin/bash
```

```
root@0aa48fea29b7:/tmp# uname -a
Linux 0aa48fea29b7 4.19.104-microsoft-standard #1 SMP Wed Feb 19 06:37:35 UTC 2020 aarch64 aarch64 aarch64 GNU/Linux
```

由于git下载太慢了并且可能中断，所以使用curl下载

```
curl -L -o zerotier-1.4.6.tar.gz --retry 20  https://github.com/zerotier/ZeroTierOne/archive/1.4.6.tar.gz
tar zxf zerotier-1.4.6.tar.gz
cd ZeroTierOne-1.4.6
```

### 清理

```
make clean
```

### 编译

```
make
```

如果需要静态编译:

```
 make SHARED=0 CC='gcc -static' CXX='g++ -static'
```

#### 查看编译结果

下面是动态编译的结果：

```
root@0aa48fea29b7:/tmp/ZeroTierOne-1.4.6# ls -lh | grep zerotier-one
lrwxrwxrwx  1 root root   12 Jun 15 05:31 zerotier-cli -> zerotier-one
lrwxrwxrwx  1 root root   12 Jun 15 05:31 zerotier-idtool -> zerotier-one
-rwxr-xr-x  1 root root 1.7M Jun 15 05:31 zerotier-one
-rw-rw-r--  1 root root 5.4K Sep  5  2019 zerotier-one.spec
```

### 查看动态编译库依赖

```
root@0aa48fea29b7:/tmp/ZeroTierOne-1.4.6# objdump -x zerotier-one | grep NEEDED
  NEEDED               libstdc++.so.6
  NEEDED               libm.so.6
  NEEDED               libgcc_s.so.1
  NEEDED               libpthread.so.0
  NEEDED               libc.so.6
  NEEDED               ld-linux-aarch64.so.1
root@0aa48fea29b7:/tmp/ZeroTierOne-1.4.6# ldd zerotier-one
        libstdc++.so.6 => /lib/aarch64-linux-gnu/libstdc++.so.6 (0x0000005501a5c000)
        libm.so.6 => /lib/aarch64-linux-gnu/libm.so.6 (0x0000005501c40000)
        libgcc_s.so.1 => /lib/aarch64-linux-gnu/libgcc_s.so.1 (0x0000005501ceb000)
        libpthread.so.0 => /lib/aarch64-linux-gnu/libpthread.so.0 (0x0000005501d10000)
        libc.so.6 => /lib/aarch64-linux-gnu/libc.so.6 (0x0000005501d40000)
        /lib/ld-linux-aarch64.so.1 (0x0000005500000000)
```

经过检查路由器上`/lib`目录，发现有除了`libstdc++.so.6`以为的库，而`libstdc++.so.6`在`/usr/lib`目录下存在。
但是运行`zerotier-one --help`之后报错：

```
./zerotier-one: error while loading shared libraries: libstdc++.so.6: cannot open shared object file: No such file or directory

```

指定从`/usr/lib`目录加载库之后仍然报错：

```
 # LD_LIBRARY_PATH=/usr/lib ./zerotier-one --help
 ./zerotier-one: error while loading shared libraries: libstdc++.so.6: wrong ELF class: ELFCLASS32
```

看起来`/usr/lib/libstdc++.so.6`是32位的。
所以最后我选择使用静态编译的结果。

### 把zerotier-one上传到路由器

用sftp工具把`zerotier-one`上传到路由器，我这里是放在`/jffs/bin`。

添加执行权限：

```
cd /jffs/bin
chmod u+x zerotier-one
```

创建链接

```
ln -sf zerotier-one zerotier-cli
ln -sf zerotier-one zerotier-idtool
```

验证是否能运行:

```
./zerotier-one --help
```

如果正确无误应该得到输出：

```
ZeroTier One version 1.4.6
Copyright (c) 2019 ZeroTier, Inc.
Licensed under the ZeroTier BSL 1.1 (see LICENSE.txt)
Usage: ./zerotier-one [-switches] [home directory]

Available switches:
  -h                - Display this help
  -v                - Show version
  -U                - Skip privilege check and do not attempt to drop privileges
  -p<port>          - Port for UDP and TCP/HTTP (default: 9993, 0 for random)
  -d                - Fork and run as daemon (Unix-ish OSes)
  -i                - Generate and manage identities (zerotier-idtool)
  -q                - Query API (zerotier-cli)
```

把`/jffs/bin`加到环境变量$PATH:

```
echo "export PATH=/jffs/bin:\$PATH" >> /jffs/configs/profile.add
```

```
ZTPID=$(pidof zerotier-one)
if [ ${ZTPID} -gt 0 ]; then
 exit 0
fi

zerotier-one -d
```

现在开始可以按照zerotier官方文档进行后续操作。

## 参考

- <https://neoautomata.com/blog/zerotier-arm/>
- <https://github.com/zerotier/ZeroTierOne>
