+++
title = "EC 协议简析"
date=2020-06-28

[taxonomies]
categories=["Programming"]
tags=["ec", "bios"]
+++

最近在用`rust`为我的`TS453B mini`写一个风扇控制程序，需要和`EC`(Embedded Controller)的交互。但是没有找到太多关于`EC`的资料。

## IO端口

`TS453B mini`使用的是以下端口：

- `0x6c` 状态/命令端口
- `0x68` 数据端口

## 协议解读

[点击这里](http://wiki.laptop.org/go/Revised_EC_Port_6C_Command_Protocol)查看协议全文。

从`Idle`状态开始总共有`8`个状态转换。这里简化为`4`条。

- 首先EC处于空闲状态，即`IBF=0`,`IOF=0`
- 第一步，检测`0x6c`，`IBF=0`时（此时最好`OBF=0`,否则会取消之前的命令和状态），向`0x6c`发送命令。这里后续可以直接转入空闲状态。
- 第二步，检测`0x6c`，直至`IBF,OBF=0,1`的时候，可以从`0x68`读取数据；如果读取后`OBF=1`，显然还需要继续读取。
- 第三步，检测`0x6c`，直至`IBF=0`的时候（此时`OBF=0`），可以向`0x68`写数据。

 总结：可以在空闲状态下安全的向`0x6c`发送命令或者向0x68写数据(触发IBF=1)。如果IBF=1，说明`EC`尚未处理发送的命令或数据。

 另外可以看到如果不加锁的话，多线程读写会有冲突的。

## 参考

- <http://wiki.laptop.org/go/Revised_EC_Port_6C_Command_Protocol>
- <http://wiki.laptop.org/go/Ec_specification#Old_Port_6c_Command_Protocol>
- <https://blog.csdn.net/zhao_longwei/article/details/50454779>
