+++
title = "Visual Studio: the project type is not supported by this installation"
date=2009-07-19

[taxonomies]
categories=["Programming"]
tags=["visual studio", "bug", "windows"]
+++
Windows 7 升级功能很好用。只不过这个过程很费时间。升级之后大部分程序都正常使用。

但是Visual Studio 2008 和 Visual Studio 2010 都罢工了。Visual Studio 2010 打不开界面，一运行就抛出异常，完全卸载后重装解决问题了。

Visual Studio 2008则是不能创建WPF项目和打不开WPF项目，创建WPF项目确定之后没有反应，打开已有的WPF解决方案，则提示“the project type is not supported by this installation”。卸载重装完全不能解决问题，还好搜索到了解决方案：

1) devenv /setup
2) regsvr32.exe "%vs80comntools%\..\IDE\projectaggregator.dll"
3) devenv /resetskippkgs

记录一下备用。

---
从我的百度空间导入