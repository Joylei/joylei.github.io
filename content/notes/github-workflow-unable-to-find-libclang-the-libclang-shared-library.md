+++
title="Github workflow: Unable to find libclang: the `libclang` shared library"
date=2021-10-09

[taxonomies]
categories=["Programming"]
tags=["github","workflow", "clang", "rust"]
+++

## The problem

It has been a while `bindgen` did not work in the github workflow for my github project.

```
   --- stderr
  Not building on Android.
  Not building on Android.
  MSVC can handle C99, compiling code as C
  BASE_C_FLAGS= -nologo -MD -Brepro /W3 /c
  BASE_CXX_FLAGS= -nologo -MD -Brepro /W3
  CMake Warning:
    Manually-specified variables were not used by the project:

      CMAKE_ASM_FLAGS
      CMAKE_ASM_FLAGS_DEBUG


  cmake build out dir: "D:\\a\\plctag-rs\\plctag-rs\\target\\debug\\build\\plctag-sys-a3fc1b7311116163\\out"
  thread 'main' panicked at 'Unable to find libclang: "the `libclang` shared library at C:\\msys64\\mingw64\\bin\\libclang.dll could not be opened: LoadLibraryExW failed"', C:\Users\runneradmin\.cargo\registry\src\github.com-1ecc6299db9ec823\bindgen-0.59.1\src/lib.rs:2117:31
  note: run with `RUST_BACKTRACE=1` environment variable to display a backtrace
```

Wasted hours to resolve it. Finally, found that the solution is that you have to install LLVM yourself.

```yaml
- name: Cache LLVM and Clang
  id: cache-llvm
  uses: actions/cache@v2
  with:
    path: ${{ runner.temp }}/llvm
    key: llvm-11.0
  if: matrix.os == 'windows-latest'
- name: Install LLVM and Clang
  uses: KyleMayes/install-llvm-action@v1
  with:
    version: "11.0"
    directory: ${{ runner.temp }}/llvm
    cached: ${{ steps.cache-llvm.outputs.cache-hit }}
  if: matrix.os == 'windows-latest'
```

Hope this saves you some time.

## References

- https://github.com/rust-lang/rust-bindgen/issues/1797
- https://github.com/actions/virtual-environments/issues/3316
- https://github.com/godot-rust/godot-rust/blob/master/.github/composite/llvm/action.yml
