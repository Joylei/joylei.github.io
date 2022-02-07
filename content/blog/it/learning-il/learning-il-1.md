+++
title = "IL学习(一)"
date=2010-05-01

[taxonomies]
categories=["Programming"]
tags=[".Net", "IL"]
+++
## 声明入口点：
```
.method void myfunc(){
    .entrypoint
}
```
程序集中只能有一个入口点，可以在方法内的任何位置

## 调用方法：
```
.method void myfunc(){
    .entrypoint
    ldstr "hello,world"
    call void [mscorlib]System.Console::WriteLine(class System.String)
    ret
}
```
ret:说明方法的结束

顺序：
    call
    返回类型
    命名空间
    方法
    参数类型

## 方法修饰属性

public/private 可访问性

hidebysig    在子类中通过方法名称或签名隐藏父类中的方法

static    有入口点的方法必须是静态的

## 创建类
```
.assembly my {}
.class private auto ansi bird
{
    .method public hidebysig static void fly() il managed
    {
        .entrypoint
        ldstr "hello,world"
        call void [mscorlib]System.Console::WriteLine(class System.String)
        ret
    }
}
```

ldstr:加载字符串到栈

auto:由运行时决定内存而不是当前程序

声明构造函数
```
.assembly my {}
.class private auto ansi bird
{
    .method public instance void .ctor() il managed
    {
        ldarg.0    //this指针
        ldstr "hello,world"
        call void [mscorlib]System.Console::WriteLine(class System.String)
        ret
    }
    .method public hidebysig static void fly() il managed
    {
        .entrypoint
        newobj instance void bird::.ctor()
        pop
        ret
    }
}
```

ldarg：加载函数传入参数到栈，类的实例方法自动传入this指针，位于参数列表中位置0

newobj:在栈顶创建新对象

pop:弹出栈顶，这里新创建的对象没有用，所以要弹出

## 声明静态构造函数
```
.assembly my {}
.class private auto ansi bird
{
    .method public instance void .ctor() il managed
    {
        ldstr ".ctor"
        call void [mscorlib]System.Console::WriteLine(string)
        ret
    }
    .method private static void .cctor() il managed
    {
        ldstr ".cctor"
        call void [mscorlib]System.Console::WriteLine(string)
        ret
    }
    .method public hidebysig static void fly() il managed
    {
        .entrypoint
        newobj instance void bird::.ctor()//创建对象
        pop
        ret
    }
}
```

## 声明成员变量
```
.assembly my {}
.class private auto ansi bird
{
    .field private int32 i
    .method public instance void .ctor() il managed
    {
        ldarg.0    //this
        ldc.i4    10//加载int32常量10到栈
        stfld int32 bird::i//栈中的值赋给成员变量i
        ret
    }
    .method public hidebysig static void fly() il managed
    {
        .entrypoint
        newobj instance void bird::.ctor()
        ldfld int32 bird::i
        call void [mscorlib]System.Console::WriteLine(int32)
        ret
    }
}
```

ldc:加载常量到栈

ldfld:加载成员变量到栈顶

stfld:将栈顶的值赋给成员变量

## 声明静态成员变量
```
.assembly my {}
.class private auto ansi bird
{
    .field private int32 i
    .field private static int32 j
    .method public instance void .ctor() il managed
    {
        ldarg.0    //this指针
        ldc.i4    20
        stsfld int32 bird::j //初始化静态成员j
        ldc.i4    10//加载int32常量10到栈
        stfld int32 bird::i//栈中的值赋给成员变量i
        ret
    }
    .method public hidebysig static void fly() il managed
    {
        .entrypoint
        newobj instance void bird::.ctor()
        //输出成员变量
        ldfld int32 bird::i
        call void [mscorlib]System.Console::WriteLine(int32)
        //输出静态成员变量
        ldsfld int32 bird::j
        call void [mscorlib]System.Console::WriteLine(int32)
        ret
    }
}
```

## 声明局部变量
```
.assembly my {}
.class private auto ansi bird
{
    .field private int32 i
    .field private static int32 j
    .method public instance void .ctor() il managed
    {
        ldarg.0    //this指针
        ldc.i4    20
        stsfld int32 bird::j //初始化静态成员j
        ldc.i4    10//加载int32常量10到栈
        stfld int32 bird::i//栈中的值赋给成员变量i
        ret
    }
    .method public hidebysig static void fly() il managed
    {
        .entrypoint
        .locals(int32 V_0,class bird V_1)//声明局部变量
        newobj instance void bird::.ctor()//创建对象
        stloc.1    //对象赋给变量V_1
        ldc.i4 5
        stloc.0    //将5赋给变量V_0
        //重新加载对象,这样才能操作它的成员和方法
        ldloc.1
        //输出成员变量
        ldfld int32 bird::i
        call void [mscorlib]System.Console::WriteLine(int32)
        //重新加载对象,这样才能操作它的成员和方法
        ldloc.1
        ldloc.0    //值
        stfld int32 bird::i//赋值
        //重新加载对象,这样才能操作它的成员和方法
        ldloc.1
        //输出成员变量
        ldfld int32 bird::i
        call void [mscorlib]System.Console::WriteLine(int32)
        //输出静态成员变量
        ldsfld int32 bird::j
        call void [mscorlib]System.Console::WriteLine(int32)
        ret
    }
}
```

局部变量自动命名为V_x的形式

stloc:把栈顶的值赋给局部变量

ldloc:把局部变量加载到栈顶

## 声明成员方法
```
.assembly my {}
.class private auto ansi bird
{
    .field private int32 i
    .method public instance void .ctor() il managed
    {
        ldarg.0    //this
        ldc.i4    10//加载int32常量10到栈
        stfld int32 bird::i//栈中的值赋给成员变量i
        ret
    }
    .method public instance void fly(string over) il managed
    {
        ldstr "fly over {0}"
        ldarg.1
        call void [mscorlib]System.Console::WriteLine(string,object)
        ret
    }
    .method public hidebysig static void main() il managed
    {
        .entrypoint
        .locals(class bird V_0)
        newobj instance void bird::.ctor()
        stloc.0
        ldloc.0
        ldfld int32 bird::i
        call void [mscorlib]System.Console::WriteLine(int32)
        ldloc.0
        ldstr "lake"
        call instance void bird::fly(string)
        ret
    }
}
```

---
从我的百度空间导入