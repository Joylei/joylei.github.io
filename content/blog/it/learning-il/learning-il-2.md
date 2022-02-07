+++
title = "IL学习(二)"
date=2010-05-01

[taxonomies]
categories=["Programming"]
tags=[".Net", "IL"]
+++

## 标签
标签能用在代码中任意跳转

虽然C#和IL中都定义了bool类型，但是他们的值却是0和1，可见这是和c、c++是一脉相承的。

br:无条件跳转

brtrue:当值为1时跳转

brfalse:当值为0时跳转

## if语句
```
.assembly my{}
.class bird
{
    .method static void main()
    {
        .entrypoint
        .locals(bool V_0)
        ldc.i4 1    //常量1
        stloc.0    //bool变量赋值1
        ldloc.0    //使用bool变量
        brfalse IL_001    //如果为假跳转到结束
        ldstr "in if statement"
        call void [mscorlib]System.Console::WriteLine(string)
        IL_001: ret
    }
}
```

## if else语句
```
.assembly my{}
.class bird
{
    .method static void main()
    {
        .entrypoint
        .locals(bool V_0)
        ldc.i4 0    //常量1
        stloc.0    //bool变量赋值1
        ldloc.0    //使用bool变量
        brfalse IL_001    //如果为假跳转到结束
        ldstr "in if true statement"
        call void [mscorlib]System.Console::WriteLine(string)
        br IL_002    //true语句完成
        IL_001:    ldstr "in if false statement"
        call void [mscorlib]System.Console::WriteLine(string)
        IL_002: ret
    }
}
```

## 算术运算符
add/sub/mul/div

```
.assembly my{}
.class bird
{
    .method static void main()
    {
        .entrypoint
        .locals(int32 V_0,int32 V_1)
        ldc.i4 12
        stloc.0
        ldc.i4 5
        stloc.1
        ldloc.0
        ldloc.1
        add    //加
        call void [mscorlib]System.Console::WriteLine(int32)
        ldloc.0
        ldloc.1
        sub    //减
        call void [mscorlib]System.Console::WriteLine(int32)
        ldloc.0
        ldloc.1
        mul    //乘
        call void [mscorlib]System.Console::WriteLine(int32)
        ldloc.0
        ldloc.1
        div    //除
        call void [mscorlib]System.Console::WriteLine(int32)
        ret
    }
}
```

## ++操作符
++操作符是+1操作
```
.assembly my{}
.class bird
{
    .method static void main()
    {
        .entrypoint
        .locals(int32 V_0)
        ldc.i4 5
        stloc.0
        ldloc.0
        ldc.i4 1
        add    //V_0++
        call void [mscorlib]System.Console::WriteLine(int32)
        ret
    }
}
```

## 关系运算符

- cgt:大于
- clt:小于
- ceq:等于
- 不等于是将ceq的结果与false再次进行ceq操作

```
.assembly my{}
.class bird
{
    .method static void main()
    {
        .entrypoint
        .locals(int32 V_0,int32 V_1,bool V_2)
        ldc.i4 5
        stloc.0
        ldc.i4 10
        stloc.1
        ldloc.0
        ldloc.1
        cgt
        stloc.2
        ldstr "5 > 10 :{0}"
        ldloca V_2
        box [mscorlib]System.Int32
        call void [mscorlib]System.Console::WriteLine(string,object)
        ldloc.0
        ldloc.1
        clt
        stloc.2
        ldstr "5 < 10 :{0}"
        ldloca V_2
        box [mscorlib]System.Int32
        call void [mscorlib]System.Console::WriteLine(string,object)
        ldloc.0
        ldloc.1
        ceq
        stloc.2
        ldstr "5 == 10 : {0}"
        ldloca V_2
        box [mscorlib]System.Int32
        call void [mscorlib]System.Console::WriteLine(string,object)
        ldloc.0
        ldloc.1
        ceq
        ldc.i4 0
        ceq
        stloc.2
        ldstr "5 != 10 :{0}"
        ldloca V_2
        box [mscorlib]System.Int32
        call void [mscorlib]System.Console::WriteLine(string,object)
        ret
    }
}
```

## while循环
循环的条件在后面

ble:当小于等于时跳转

```
.assembly my{}
.class bird
{
    .method static void main()
    {
        .entrypoint
        .locals(int32 V_0)
        ldc.i4 0
        stloc.0
        br IL_002
        ldloc.0
        call void [mscorlib]System.Console::WriteLine(int32)
        ldloc.0
        ldc.i4 1
        add
        stloc.0 //V_0++
        IL_002: ldloc.0
        ldc.i4 5
        ble IL_001 //V_0 <= 5
        ret
    }
}
```

## do...while循环
```
.assembly my{}
.class bird
{
    .method static void main()
    {
        .entrypoint
        .locals(int32 V_0)
        ldc.i4 0
        stloc.0
        IL_001: ldloc.0
        call void [mscorlib]System.Console::WriteLine(int32)
        ldloc.0
        ldc.i4 1
        add
        stloc.0 //V_0++

        ldloc.0
        ldc.i4 5
        ble IL_001 //V_0 <= 5
        ret
    }
}
```

## for循环
```
.assembly my{}
.class bird
{
    .method static void main()
    {
        .entrypoint
        .locals(int32 V_0)
        ldc.i4 0
        stloc.0
        br IL_002
        IL_001: ldloc.0
        call void [mscorlib]System.Console::WriteLine(int32)
        ldloc.0
        ldc.i4 1
        add
        stloc.0 //V_0++
        IL_002: ldloc.0
        ldc.i4 5
        ble IL_001 //V_0 <= 5
        ret
    }
}
```

## break/continue
```
.assembly my{}
.class bird
{
    .method static void main()
    {
        .entrypoint
        .locals(int32 V_0)
        ldc.i4 0
        stloc.0
        br IL_002   
        IL_001: ldloc.0
        ldc.i4 1
        add
        stloc.0 //V_0++
        ldloc.0
        ldc.i4 2
        div
        ldc.i4 1
        ceq
        brtrue IL_002 //continue
        ldloc.0
        ldc.i4 3
        ceq
        brtrue IL_003    //break
        ldloc.0
        call void [mscorlib]System.Console::WriteLine(int32)
        IL_002: ldloc.0
        ldc.i4 5
        ble IL_001 //V_0 <= 5
        IL_003: ret
    }
}
```

---
从我的百度空间导入