+++
title = "《Programming F#》读书笔记六"
date=2010-03-09

[taxonomies]
categories=["Reading"]
tags=["Programming", "F#"]
+++
## 索引
- [《Programming F#》读书笔记一](@/blog/life/reading/programming-fsharp/programming-fsharp-1.md)
- [《Programming F#》读书笔记二](@/blog/life/reading/programming-fsharp/programming-fsharp-2.md)
- [《Programming F#》读书笔记三](@/blog/life/reading/programming-fsharp/programming-fsharp-3.md)
- [《Programming F#》读书笔记四](@/blog/life/reading/programming-fsharp/programming-fsharp-4.md)
- [《Programming F#》读书笔记五](@/blog/life/reading/programming-fsharp/programming-fsharp-5.md)
- [《Programming F#》读书笔记六](@/blog/life/reading/programming-fsharp/programming-fsharp-6.md)
- [《Programming F#》读书笔记七](@/blog/life/reading/programming-fsharp/programming-fsharp-7.md)
- [《Programming F#》读书笔记八](@/blog/life/reading/programming-fsharp/programming-fsharp-8.md)
- [《Programming F#》读书笔记九](@/blog/life/reading/programming-fsharp/programming-fsharp-9.md)
- [《Programming F#》读书笔记十](@/blog/life/reading/programming-fsharp/programming-fsharp-10.md)
- [《Programming F#》读书笔记十一](@/blog/life/reading/programming-fsharp/programming-fsharp-11.md)
- [《Programming F#》读书笔记十二](@/blog/life/reading/programming-fsharp/programming-fsharp-12.md)

## 笔记
Computation Expression:

作用：改变代码执行逻辑；减少重复代码；扩展F#语言

限制：
- 1.不能在Computation Expression中定义新类型
- 2.不能在Computation Expression中使用mutable变量，用ref变量代替


下面定义一个允许使用let!和return关键字的computation expression
```f#
type DefinedBuilder() =
    member this.Bind((x:int),(rest:int->int)) = //允许使用let!
        printfn "x:%d" x
        match x with
        | x when x>0 ->rest x
        | _ -> x
    member this.Return(x:'a) = x //允许使用return

let defined = DefinedBuilder()

let expr a b c =
    defined{
        let! x = a *2 //只有当x>0时才会执行以下剩余语句，否则返回x
        let! y = b - 5//只有当y>0时才会执行以下剩余语句，否则返回y
        let! z = c % 3//只有当z>0时才会执行以下剩余语句，否则返回z
        return x + y + z
    }

printfn "expr -2 3 5 = %d" (expr -2 3 5) //返回x
printfn "expr 1 3 5 = %d" (expr 1 3 5) //返回y
printfn "expr 1 6 3 = %d" (expr 1 6 3) //返回z
printfn "expr 1 6 5 = %d" (expr 1 6 5) //返回x+y+z
```

Computation Expresssion所有方法：
    方法 描述

    member For:
    seq<'a> * ('a -> Result<unit>) ->
    Result<unit>
    允许使用for循环，参数是循环的所有数据项和循环的主体代码

    member Zero:
    unit -> Result<unit>
    允许执行unit表达式

    member Combine:
    Result<unit> * Result<'a> ->
    Result<'a>
    把computation expression各部分连接起来

    member While:
    (unit -> bool) * Result<unit> ->
    Result<unit>
    允许使用while循环

    member Return:
    'a -> Result<'a>
    允许使用return关键字

    member ReturnFrom:
    'a -> Result<'a>
    允许使用return!关键字

    member Yield:
    'a -> Result<'a>
    允许使用yield关键字

    member YieldFrom:
    seq<'a> -> Result<'a>
    允许使用yield!关键字

    member Delay:
    (unit -> Result<'a>) ->
    Result<'a>
    这个操作通常和Combine一起使用，以确保操作以正确的顺序执行

    member Run:
    Result<'a> -> Result<'a>
    在Computation Expression的开始执行时被调用

    member Using:
    'a * ('a -> Result<'b>) ->
    Result<'b> when 'a :>
    System.IDisposable
    允许使用use!关键字，负责IDisposable.Dispose()

    member Bind:
    Result<'a> * ('a -> Result<'b>) ->
    Result<'b>
    允许使用let!关键字(do!是let!的特殊形式，Bind返回

    Result<unit>)

    member TryFinally:
    Result<'a> * (unit -> unit) ->
    Result<'a>
    允许使用try finally表达式，参数:try代码块的结果和finally代码块代表的函数

    member TryWith:
    Result<'a> * (exn -> Result<'a>) ->
    Result<'a>
    允许使用try with表达式，参数:try代码块的结果和with代码块代表的函数 


---
从我的百度空间导入