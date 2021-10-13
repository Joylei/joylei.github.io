+++
title = "qute 代码发布到github"
date=2020-09-07

[taxonomies]
categories=["Programming"]
tags=["qute", "QNAP", "TS453B mini", "Fan Control", "Reverse Engineering", "rust", "radare"]
+++

今天把我为[`qute`](https://github.com/Joylei/qute)发布到了`github`。

## 为什么

我有一台`TS453B mini`，但是说实话用的并不是很顺畅，系统特别不稳定，总是容易卡死。想着换成其它系统，最后选择了`Alpine Linux`。
虽然遇到了一些困难，总算是解决了，顺利运行起来。但是有个问题一直无解，那就是没法控制风扇的转速。在`BIOS`中可以设置风扇为`auto`或者`manual`。
实测`auto`不靠谱，机器会热死机。设置`manual`的话，风扇在机器启动时以100%转速运转，如果可以忍耐这个噪音的话，到这一步就可以了。
那么能不能直接控制风扇呢？没有任何头绪，直到发现了<https://github.com/guedou/TS-453Be>，然后折腾一番，没有在`QTS`中运行起来，
报了`segment fault`错误。作者提供了完整的步骤和思路，我就想自己写一个吧，大概是从6、7月份开始的，用`radare2`进行逆向分析，然后就有了[qute](https://github.com/Joylei/qute)。期间还想了很多，例如是不是写成驱动形式，是不是写一个自动控制调整转速的逻辑，是不是支持获取其它设备的温度；所以看了怎么实现`Linux`驱动和`Linux`驱动相关实现代码，怎么用`rust`写`Linux`驱动，硬盘`SMART`属性读取等等。写成驱动形式好处就是可以配合其它`PWM`调度控制的程序，不用自己去写控制逻辑。<s>想了很多但最后都放弃了没有实现。</s>2021年7月实现了自动调试控制。

## 是什么

`qute`是用`rust`写的通过`EC`协议来控制`QNAP NAS`的风扇转速。理论上只要是基于`ITE8528`的`QNAP NAS`都可以控制。
除了控制风扇和获取温度，还可以设置`EuP`, `Power Recovery Mode`。

关于`EC`协议，请查看我的另一片文章[EC 协议简析](@/blog/it/ec-protocol.md)。

## 怎么用

注意：需要`root`权限运行。

```text
qute v0.1

QNAP device control. Use AT YOUR OWN RISK!!!

USAGE: qute [OPTIONS] [COMMANDS]

OPTIONS:
  -V, --version                 Show version number
  -h, --help                    Show help message
  -v, --verbose [level:N]       Show verbose messages
  -q, --quiet                   Silence all output

COMMANDS:
  eup                           get or set Eup mode
  fan                            get or set fan speed
  power                       get or set power recovery mode
  temp                         get temperature
```

## TS-453B mini 规格信息
```
453Bmini

Intel Celeron J3455 CPU is a 1.5GHz frequency, that can be bursted to 2.3GHz per core

HD Graphics 500 GPU with 12 EUs

DDR3L 1600 RAM

风扇 fan: xtreme BD121232LB  元山科技

(120 mm, 12 V,1700 RPM, 23 CFM, 42.5 dBA, and 80000 MTBF)

+12V DC
4PIN (PWM)
120 x 120 x 32mm
1500 min^-1 nominal
Cable length 250 mm
Mounting holes 105 x 105 +/0.3 mm
0.19A
```
内存可以加到16GB。

## 更多

请移步[`github`](https://github.com/Joylei/qute)了解更多。

## 参考

- https://github.com/guedou/TS-453Be