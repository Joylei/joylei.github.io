+++
title="Profiling rust on windows with Luke Stackwalker"
date=2020-12-24

[taxonomies]
categories=["Programming"]
tags=["rust", "bindgen", "clang"]
+++

There are great profiling tools for rust, such as [profiling](https://crates.io/crates/profiling), [gperftools](https://crates.io/crates/gperftools), [flamegraph](https://crates.io/crates/flamegraph), etc.  But I would recommend to try
[Luke Stackwalker](https://sourceforge.net/p/lukestackwalker) which also works with rust.

> [Luke Stackwalker](https://sourceforge.net/p/lukestackwalker) is a GUI-based C/C++ source profiler for windows.It samples your application's stack while the application is running to find out where the application spends most of its time.

 To get start, you need to turn on debug info for rust.

```toml
[profile.release]
debug=1
```

 And start `Luke Stackwalker` with administrator privilege, otherwise it would crash.

When I tried it on my project, everything looks good at first. Most cpu time was cost on `WaitForSingleObject`, it's totally fine, because the code is waiting for incoming http requests. Then I noticed there were abnormally file open calls on the top list, which should be very very low. I found that I mistakenly open the log file every time there is a write operation.

Have a try.
