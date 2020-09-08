+++
title = "WSL ubuntu add user"
date=2020-06-14


[taxonomies]
categories=["Windows"]
tags=["windows", "wsl", "ubuntu"]
+++

## 打开命令行，以root用户运行wsl

```
C:\WINDOWS\system32>wsl -u root
```

## 添加 user

```
root@MYPC:/mnt/c/WINDIWS/system32# adduser lei
```

按照提示输入用户名和密码

## 给新用户添加sudo权限

```
root@MYPC:/mnt/c/WINDIWS/system32# usermod -G sudo lei
```

## 退出当前wsl

```
exit
```

## 切换默认用户

```
C:\WINDOWS\system32>ubuntu2004 config --default-user lei
```

这里ubuntu版本是20.04，如果是ubuntu 18.04，那么运行的是`ubuntu1804 config --default-user <username>`；以此类推，如果是其它发行版，改为相应的发行版名称+版本号。

## 启动新的wsl，验证新用户和sudo是否有效

```
C:\WINDOWS\system32>wsl
lei@MYPC:/mnt/c/WINDIWS/system32#sudo apt update
[sudo] password for lei:
Get:1 http://archive.ubuntu.com/ubuntu focal InRelease [265 kB]
Get:2 http://security.ubuntu.com/ubuntu focal-security InRelease [107 kB]
Get:3 http://security.ubuntu.com/ubuntu focal-security/main amd64 Packages [103 kB]
Get:4 http://security.ubuntu.com/ubuntu focal-security/main Translation-en [38.5 kB]
Get:5 http://archive.ubuntu.com/ubuntu focal-updates InRelease [107 kB]
```

## 参考

- <https://docs.microsoft.com/zh-cn/windows/wsl/wsl-config>
- <https://www.tenforums.com/tutorials/128052-add-user-windows-subsystem-linux-wsl-distro-windows-10-a.html>
- <https://www.thewindowsclub.com/how-to-set-default-user-switch-user-and-remove-a-user-for-wsl/>
