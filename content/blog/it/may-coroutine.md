+++
title = "May -  rust stackful coroutine"
date=2020-09-06

[taxonomies]
categories=["Programming"]
tags=["rust", "may", "coroutine"]
+++

最近在项目中尝试了`rust`下的协程库[`May`](https://github.com/Xudong-Huang/may)。如果你对写`async\await`感到厌倦，可以试试`May`,换换脑子。

## 介绍

[`May`](https://github.com/Xudong-Huang/may)是一个用`pure rust`写的基于栈的协程库。官方的介绍如下：

> May is a high-performant library for programming stackful coroutines with which you can easily develop and maintain massive concurrent programs. It can be thought as the Rust version of the popular Goroutine.

常见的协程库，在`python`下有`gevent`, `c\c++`有`libuv`。在`rust`下也有其它基于`ffi`封装的`libuv`等库。

## 和`Future`简单对比

`May`和`rust`的`Future`有哪些相同和不同的地方？

相同的地方：

- 都是基于`generator`实现的，也就是说都是`stackful`
- 捕获的变量都需要`move`
- 都需要`runtime`去执行

不同的地方：

- `Future`需要`pin reference`, `May`不需要(可能也需要？)

- `Future`使用`async\await`关键字，`May`不需要

## 示例

下面的示例简单展示了`task`的`spawn`和`cancel`。

```rust
#[macro_use]
extern crate may;

use may::coroutine as co;
use std::time::{Duration, Instant};
fn main() {
    let t = go!(|| main_task());
    t.join();
}

fn main_task() {
    let mut tasks = vec![];
    for i in 0..10 {
        let task = go!(move || { child_task(i) });
        tasks.push(task);
    }
    co::sleep(Duration::from_secs(3));
    for (i, t) in (&tasks[0..5]).iter().enumerate() {
        unsafe {
            t.coroutine().cancel();
            println!("child {}: cancelled", i);
        }
    }
    for t in tasks {
        t.join();
    }
    println!("all done!");
}

fn child_task(index: usize) {
    let now = Instant::now();
    loop {
        if now.elapsed() > Duration::from_secs(20) {
            println!("child {}: done", index);
            break;
        }
        println!("child {}: running", index);
        co::sleep(Duration::from_millis(500));
    }
}

```

## Cancel 的问题

使用`cancel`后发现程序在`release`模式下莫名的崩溃。
调试及查看源代码，发现是由于`May`的[cancel](https://github.com/Xudong-Huang/may/blob/master/src/cancel.rs)机制使用`rust`的`panic`机制。所以无法在编译时优化使用自定义的`panic`处理机制。

```toml
[profile.release]
panic = "abort"
```

如果使用这个配置，`cancel()`时应用程序会崩溃退出。

进一步阅读`may`的代码发现，`may`不会主动`panic`，`panic`是由于其它代码`panic`被它捕捉到了。可能是哪里`unwrap`之类的调用抛出的。
