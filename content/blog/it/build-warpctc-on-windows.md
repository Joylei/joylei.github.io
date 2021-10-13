+++
title="Build WarpCTC on Windows"
date=2020-11-09

[taxonomies]
categories=["Programming"]
tags=["CTC", "python"]
+++

需要用到`warpctc pytorch binding`，又不想切换到`linux`平台，于是修改一番，添加了一键编译脚本。

在编译时可能会遇到CUDA报错，请修改`CMakeLists.txt`，在里面注释或者反注释CUDA对应的架构。

修改后的代码已发布在[Github](https://github.com/Joylei/warp-ctc)。
