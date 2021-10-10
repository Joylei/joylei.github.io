+++
title = "在Windows下编译运行EJDB2"
date=2021-01-22

[taxonomies]
categories=["Programming"]
tags=["mingw", "ejdb", "nosql","windows"]
+++

最近在给[EJDB2](http://ejdb.org)写[`rust`的绑定](https://github.com/Joylei/ejdb2-rs)。本文记录怎么在 Windows 下编译及正常运行`EJDB2`。
直接到文章末尾看结论。

## 安装环境

参考官方文档安装`msys2`、`mingw64`。
安装开发工具链：

```
pacman -S --needed base-devel git vim \
      mingw-w64-x86_64-toolchain \
```

## 编译

```sh
cmake .. -DCMAKE_C_COMPILER=gcc -DCMAKE_BUILD_TYPE=Release -DCMAKE_INSTALL_PREFIX=./output -DBUILD_SHARED_LIBS=OFF -DENABLE_HTTP=OFF
make
make install
```

直接编译不通过。

- `mmap`的问题
  `iowow`项目中本身有`mman`相关代码`platform\win32\mman`，并且配置了一个`CMakeLists.txt`来导入`mman/mman.c`源代码，但是这个`CMakeLists.txt`没有生效，导致不能编译。
  在`platform\win32\win32.c`中添加

```c
#include "mman/mman.c"
```

- 找不到`localtime_r`
  将`log\iwlog.c`文件中的`localtime_r(&ts_sec, &timeinfo)`修改为`localtime_s(&timeinfo,&ts_sec)`

```c
#ifndef _WIN32
  localtime_r(&ts_sec, &timeinfo);
#else
  localtime_s(&timeinfo, &ts_sec);
#endif
```

注意：`localtime_s`和`localtime_r`的参数的顺序是相反的。

- `strndup`的问题
  链接时加上`iberty`依赖，不需要改代码。如果不想加上`iberty`依赖，自己定义一个`strndup`:

```c
char *strndup(const char *src, size_t len)
{
  char *dst = NULL;
  if (src)
  {
    size_t src_len = strlen(src);
    int count = min(src_len, max(len, 0));
    dst = malloc(count + 1);
    if (dst)
    {
      strncpy(dst, src, count);
      dst[count] = '\0';
    }
  }
  return dst;
}
```

做完以上修改，应该可以编译通过了。

## 修复运行报错

运行编译好的可执行程序，运行报错：

```
ERROR iwkv.c:3590 70007|22|0|Threading error with errno status set. (IW_ERROR_THREADING_ERRNO)|Invalid argument
```

`example`代码一样报错。

虽然可以知道是哪个文件报错，但是 rc 返回值可能来自很多地方，看代码不能直接定位到具体出错的地方。`70007`在代码中代表`pthread`相关错误，所以应该是`pthread`某个函数调用出错了。

搜索之后发现这个错误`Invalid argument`是函数验证参数时抛出的错误。
于是想用`_set_invalid_parameter_handler`来`hook`出错的位置。代码如下：

```rs
use libc::c_void;

type FPHandler = Option<
    unsafe extern "C" fn(
        expr: *const u16,
        func: *const u16,
        file: *const u16,
        line: u32,
        pReserved: c_void,
    ),
>;

extern "system" {
    fn _set_invalid_parameter_handler(fp: FPHandler);
}
unsafe fn u16_ptr_to_string(ptr: *const u16) -> OsString {
    use std::os::windows::ffi::OsStringExt;
    let len = (0..).take_while(|&i| *ptr.offset(i) != 0).count();
    let slice = std::slice::from_raw_parts(ptr, len);

    OsString::from_wide(slice)
}
unsafe extern "C" fn my_handler(
    expr: *const u16,
    func: *const u16,
    file: *const u16,
    line: u32,
    pReserved: c_void,
) {
    use std::ffi::{OsStr, OsString};

    let expr = u16_ptr_to_string(expr);
    let func = u16_ptr_to_string(func);
    let file = u16_ptr_to_string(file);
    println!("expr: {}", expr.to_str().unwrap_or_default());
    println!("func: {}", func.to_str().unwrap_or_default());
    println!("file: {}", file.to_str().unwrap_or_default());
    println!("line: {}", line);
    panic!("invalid parameter")
}

//hook
unsafe {
    _set_invalid_parameter_handler(Some(my_handler));
}
```

但是没有起作用。

试了`GDB` 调试，不太会用，放弃了。

接下来使用`IDA`单步调试，跟踪到具体函数为`rci = pthread_condattr_setclock(&cattr, CLOCK_MONOTONIC)`。
查看[pthread_condattr_setclock 的实现](https://github.com/msys2-contrib/mingw-w64/blob/78dca70d9c2c355f0aaea3292e0d68c57ad7a5c5/mingw-w64-libraries/winpthreads/src/cond.c)，发现第二个参数`clock_id`只接受`0`，而在`iowow`代码中有定义`#define CLOCK_MONOTONIC 1`。最简单的做法，在这里条件判断一下传入`0`。但是根本原因是`CMakeLists.txt`判断`HAVE_CLOCK_MONOTONIC`出问题了，判断的相关代码：

```cmake
check_symbol_exists(CLOCK_MONOTONIC time.h HAVE_CLOCK_MONOTONIC)
if (HAVE_CLOCK_MONOTONIC)
  add_definitions(-DIW_HAVE_CLOCK_MONOTONIC)
endif()
```

`time.h`里面确实没有`CLOCK_MONOTONIC`，不明白哪里有问题。改成下面的代码：

```cmake
check_symbol_exists(CLOCK_MONOTONIC time.h HAVE_CLOCK_MONOTONIC)
if (HAVE_CLOCK_MONOTONIC and WIN32)
  add_definitions(-DIW_HAVE_CLOCK_MONOTONIC)
  message(INFO "HAVE_CLOCK_MONOTONIC=1")
endif()
```

## 以上白做了

最后发现以上白做了，只需要安装`mingw-w64-x86_64-cmake`：

```
pacman -S --needed base-devel git vim \
      mingw-w64-x86_64-toolchain \
      mingw-w64-x86_64-cmake
```

然后在`mingw64`的 shell 执行：

```sh
cmake .. -G "MSYS Makefiles" -DCMAKE_C_COMPILER=gcc -DCMAKE_BUILD_TYPE=Release -DCMAKE_INSTALL_PREFIX=./output -DBUILD_SHARED_LIBS=OFF -DENABLE_HTTP=OFF
make
make install
```

不需要改动任何代码就可以正常编译和运行。

## 静态链接

```sh
cmake .. -G "MSYS Makefiles" -DCMAKE_C_COMPILER=gcc -DCMAKE_BUILD_TYPE=Release -DCMAKE_INSTALL_PREFIX=./output -DBUILD_SHARED_LIBS=OFF -DENABLE_HTTP=OFF
make
make install
```

自己的项目静态链接需要链接以下库文件：

```
ejdb2-2
iowow-1
iberty
winpthread
mingwex
mingw32
gcc
msvcrt
```

## 动态链接

- 修改 `cmake/modules/AddIOWOW.cmake`， 把`-DBUILD_SHARED_LIBS=OFF` 改为`-DBUILD_SHARED_LIBS=${BUILD_SHARED_LIBS}`。

- 使用自定义`toolchain`，用`mingw64`里的工具导出`dll`和相关定义文件

```cmake
#my-win64-tc.cmake
if (WIN32)
    add_custom_target(wintools_init)

    macro(add_w32_importlib tgt libname wdir)
    add_custom_command(
        TARGET ${tgt}
        POST_BUILD
        COMMAND dlltool.exe -d ${libname}.def  -e ${libname}.exp -l ${libname}.lib -D ${libname}.dll
        WORKING_DIRECTORY ${wdir}
    )
    endmacro(add_w32_importlib)
endif()
```

- 编译

```sh
cmake .. -G "MSYS Makefiles" -DCMAKE_C_COMPILER=gcc -DCMAKE_BUILD_TYPE=Release -DCMAKE_INSTALL_PREFIX=./output -DBUILD_SHARED_LIBS=ON -DENABLE_HTTP=OFF -DCMAKE_TOOLCHAIN_FILE=my-win64-tc.cmake
make
make install
```

自己的项目动态链接需要链接以下库文件：

```
libejdb2
libwow
```

## 最后

ejdb2 rust binding: <https://github.com/Joylei/ejdb2-rs>

## 参考

- <https://www.cnblogs.com/oloroso/p/12632412.html>
- <https://github.com/Softmotions/ejdb/blob/master/WINDOWS.md>
