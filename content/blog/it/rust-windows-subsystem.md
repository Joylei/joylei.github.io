+++
title = "About rust windows subsystem"
date=2022-01-17

[taxonomies]
categories=["Programming"]
tags=["rust", "windows", "gui"]
+++

Learnt that it need to configure additional attribute for GUI application on windows platform. Otherwise it will launch a console window while launching your GUI application.

```rust
#![windows_subsystem = "windows"]
```

Turns on this feature, you'll encounter a strange issue. There is no backtrace anymore when your application panics, even though you have configured the `RUST_BACKTRACE` environment. You will only get this information:
```sh
error: process didn't exit successfully: `target\debug\XXXXXX.exe` (exit code: 101)
```
It's difficult to trace what happened in this situation.

No luck with newest version and nightly rust.
```sh
> rustc --version --verbose
rustc 1.58.0 (02072b482 2022-01-11)
binary: rustc
commit-hash: 02072b482a8b5357f7fb5e5637444ae30e423c40
commit-date: 2022-01-11
host: x86_64-pc-windows-msvc
release: 1.58.0
LLVM version: 13.0.0
```

## references

- https://rust-lang.github.io/rfcs/1665-windows-subsystem.html
- https://github.com/rust-lang/rust/issues/88576
