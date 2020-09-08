+++
title="python pip source"
date=2020-07-26

[taxonomies]
categories=["Programming"]
tags=["python","pip"]
+++

修改pip默认源：

```
pip install -i http://pypi.douban.com/simple/ --trusted-host=pypi.douban.com/simple ipython
```

```
pip config set global.index-url https://mirrors.aliyun.com/pypi/simple

```