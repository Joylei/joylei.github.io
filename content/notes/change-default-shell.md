+++
title="Linux修改默认shell"
date=2020-07-15

[taxonomies]
categories=["Linux"]
tags=["linux","shell"]
+++

## 查看支持的shell

```
cat /etc/shells
```

## 修改shell

```
cat /etc/passwd
```

输出类似于：

```
root:x:0:0:root:/root:/bin/ash
bin:x:1:1:bin:/bin:/sbin/nologin
daemon:x:2:2:daemon:/sbin:/sbin/nologin
adm:x:3:4:adm:/var/adm:/sbin/nologin
......
```

修改`root:x:0:0:root:/root:/bin/ash`最后一部分的`/bin/ash`为shell的路径，然后保存就可以了。
