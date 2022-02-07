+++
title = "IL学习(五)"
date=2010-05-22

[taxonomies]
categories=["Programming"]
tags=[".Net", "IL"]
+++

## 指针

- `&`用于托管的指针
- `*`用于非托管的指针

## 非托管指针
    这个概念借鉴了c和c++
    使用没有限制
    被执行引擎识别为无符号数
    不被垃圾回收器管理

指针变量是一个无符号数

```
.assembly my{}
.class test
{
    .method public static void main() il managed
    {
        .entrypoint
        .locals(int32 *V_0,int32 V_1)
        ldc.i4 20
        stloc V_0   //将int类型的数值存放在指针变量中
        ldloc V_0   //加载数据
        call void [mscorlib]System.Console::WriteLine(int32)
        ldloca V_0 //获取V_0地址
        stloc V_1   //存放在V_1中
        ldloc V_1
        call void [mscorlib]System.Console::WriteLine(int32)
        ret
    }
}
```

## 通过指针加载变量
```
.assembly my{}
.class test
{
    .method public static void main() il managed
    {
        .entrypoint
        .locals(int32 V_0)
        ldc.i4 20
        stloc V_0
        ldloca V_0 //指针地址
        ldobj int32 //加载地址指向的变量
        call void [mscorlib]System.Console::WriteLine(int32)
        ret
    }
}
```

ldobj:从栈顶存放的地址加载变量到栈顶

所有的局部变量都创建在栈上，其它的创建在堆上。栈中只能存放数字。
```
.assembly my{}
.class test
{
    .method public static void main() il managed
    {
        .entrypoint
        .locals(int32 V_0)
        ldstr "测试1"
        call void [mscorlib]System.Console::WriteLine(int32) //输出栈顶的字符串的地址
        ldstr "测试2"
        stloc V_0
        ldloca V_0
        ldobj int32
        call void [mscorlib]System.Console::WriteLine(int32)
        ret
    }
}

.assembly my{}
.class test
{
    .field int32 i
    .field static int32 j
    .method public static void main() il managed
    {
        .entrypoint
        newobj void test::.ctor()
        ldflda int32 test::i //载入i的地址
        call void [mscorlib]System.Console::WriteLine(int32)
        ldsflda int32 test::j //载入j的地址
        call void [mscorlib]System.Console::WriteLine(int32)
        ret
    }
    .method instance void .ctor()
    {
    }
}
```

## transient指针
这是介于托管和非托管之间的指针，它只能由执行引擎创建。

---
从我的百度空间导入