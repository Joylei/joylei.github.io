+++
title="linux prefer ipv4 other than ipv6"
date=2020-07-15

[taxonomies]
categories=["Linux"]
tags=["linux","ipv4", "ipv6", "network"]
+++
双栈环境下，Linux怎么设置IPv4优先于IPv6？
由于中国大陆很多情况下网络访问IPv6慢于IPv4，所以最好设置IPv4优先。

## How

`/etc/gai.conf`
如果没有这个文件就创建这个文件，添加或修改以下内容
```
precedence ::ffff:0:0/96 100
```

## References

<http://sf-alpha.bjgang.org/wordpress/2012/08/linux-prefer-ipv4-over-ipv6-in-dual-stack-environment-and-prevent-problems-when-only-ipv4-exists/>