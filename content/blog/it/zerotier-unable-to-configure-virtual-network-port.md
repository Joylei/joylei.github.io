+++
title = "zerotier: unable to configure virtual network port"
date=2020-06-22

[taxonomies]
categories=["Network"]
tags=["zerotier", "network"]
+++

```
ERROR: 
unable to configure virtual network port: could not open TUN/TAP device: No such file or directory
```

Solution:
```shell
modprobe tun
```