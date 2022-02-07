+++
title = "IL学习(四)"
date=2010-05-17

[taxonomies]
categories=["Programming"]
tags=[".Net", "IL"]
+++

## 接口限制

- 接口中所有方法必须是abstract或static
- 虚方法必须是abstract和public
- 不能有实例字段
- 接口不能实例化
- 接口不能从类继承

直接使用call调用接口方法和使用callvirt动态调用的效果是一样的。

## 结构体

il不直接支持结构体
引用类型从Object类继承，值类型从ValueType继承
```
.assembly my{}
.class value point
{
    .field int32 x
    .field int32 y
    .method static void main() il managed
    {
        .entrypoint
        .locals(value class point V_0,value class point V_1)
        ldloca V_0
        initobj value class point
        ldloca V_0
        ldfld int32 point::x
        call void [mscorlib]System.Console::WriteLine(int32)

        ldloca V_1
        initobj value class point
        ldloca V_1
        ldc.i4 10
        ldc.i4 20
        call instance void point::.ctor(int32,int32)
        ldstr "point:x={0},y={1}"
        ldloca V_1
        ldfld int32 point::x
        ldloca V_1
        ldfld int32 point::y
        call void [mscorlib]System.Console::WriteLine(string,object,object)
        ret
    }
    .method instance void .ctor(int32 x,int32 y) il managed
    {
        ldarg.0
        ldarg.1
        stfld int32 point::x
        ldarg.0
        ldarg.2
        stfld int32 point::y
        ret
    }
}
```
initobj:用于初始化结构体的字段为0值，它不会调用结构体的构造函数。对于结构体初始化，不用initobj也可以，这样各个字段就得到一个随机值。

结构体也可以有static、virtual和instance方法
《c# to il》书上说值类型在调用virtual方法时只能使用call，不能使用callvirt。但是经验证是可以的，可能是.net4中改变了规则。
```
.assembly my {}
.class interface IFly
{
    .method public abstract instance void fly() il managed{}
}
.class value private auto ansi bird implements IFly
{
    .field private int32 i
    .method public instance void .ctor() il managed
    {
        ldarg.0    //this
        ldc.i4    10
        stfld int32 bird::i
        ret
    }
    .method virtual public instance void fly() il managed
    {
        ldstr "fly"
        ldarg.1
        call void [mscorlib]System.Console::WriteLine(string,object)
        ret
    }
    .method public hidebysig static void main() il managed
    {
        .entrypoint
        .locals(value class bird V_0,class IFly V_1)
        ldloca V_0
        call instance void bird::.ctor()
        ldloca V_0
        callvirt instance void bird::fly() //运行时决定
        ldloca V_0
        box bird
        //castclass IFly
        stloc.1
        ldloc.1
        callvirt instance void IFly::fly()
        ret
    }
}
```

## 装箱与拆箱

装箱：将值类型转换成引用类型进行的操作
拆箱：将引用类型转换成值类型进行的操作
```
.assembly my{}
.class value auto ansi sealed point
{
    .field int32 x
    .field int32 y
    .method static void main() il managed
    {
        .entrypoint
        .locals(value class point V_0,object V_1,value class point V_2)
        ldloca V_0
        ldc.i4 10
        ldc.i4 20
        call instance void point::.ctor(int32,int32) //调用构造函数
        ldloca V_0
        call instance void point::print()
        ldloca V_0
        box point //装箱成object对象,栈顶放入对象地址
        stloc.1
        ldloc.1
        unbox point //拆箱，得到新结构体的地址
        ldobj point //根据地址装载对象
        stloc.2
        ldloca V_2
        call instance void point::print()
        ret
    }
    .method instance void .ctor(int32 x,int32 y) il managed
    {
        ldarg.0 //this
        ldarg.1
        stfld int32 point::x
        ldarg.0
        ldarg.2
        stfld int32 point::y
        ret
    }
    .method public instance void print() il managed
    {
        ldstr "point:x={0},y={1}"
        ldarg.0 //this
        ldfld int32 point::x
        box int32 //装箱
        ldarg.0
        ldfld int32 point::y
        box int32
        call void [mscorlib]System.Console::WriteLine(string,object,object)
        ret
    }
}
```

- box:装箱
- unbox:拆箱
- ldobj:从地址加载对象

---
从我的百度空间导入