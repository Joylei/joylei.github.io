+++
title="linux prefer ipv4 other than ipv6"
date=2020-07-15

[taxonomies]
categories=["Linux"]
tags=["ipv4", "ipv6", "network", "setting"]
+++
双栈环境下，怎么设置IPv4优先于IPv6？
由于中国大陆很多情况下网络访问IPv6慢于IPv4，所以最好设置IPv4优先。

## Linux

`/etc/gai.conf`
如果没有这个文件就创建这个文件，添加或修改以下内容
```
precedence ::ffff:0:0/96 100
```

## Windows

### 查看当前`prefixpolicy`

```sh
netsh interface ipv6 show prefixpolicies 
```
可以看到`::ffff:0:0/96`对应的优先级和标签。


### 修改`prefixpolicy`

以管理员权限运行：
```sh
netsh interface ipv6 set prefixpolicy ::ffff:0:0/96 100 4
```
命令中`100`代表优先级， 越大优先级越低。


## References
- http://sf-alpha.bjgang.org/wordpress/2012/08/linux-prefer-ipv4-over-ipv6-in-dual-stack-environment-and-prevent-problems-when-only-ipv4-exists/
- https://bbs.pcbeta.com/viewthread-1853059-2-1.html