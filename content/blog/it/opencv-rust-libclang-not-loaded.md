+++
title="opencv-rust: libclang not loaded"
date=2020-09-16

[taxonomies]
categories=["Programming"]
tags=["opencv", "bindgen", "clang", "rust", "windows"]
+++

I failed to build a project when use `opencv-rust` as dependency on Windows 10. The error message said

```text
thread 'main' panicked at 'a `libclang` shared library is not loaded on this thread'
```

It's a little strange. Because `libclang.dll` do exist and  can be found in `PATH` env, and other projects that depends on `bindgen` can be built successfully.

Finally, found that I have to enable `clang-runtime`  feature to get the project compiled.

```toml
opencv = { version="*", features=["buildtime-bindgen", "clang-runtime"]}
```
