+++
title = "《Programming F#》读书笔记十"
date=2010-06-27

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
模式匹配

    模式匹配与switch语句类似，但是它的功能更强大，它是由一系列规则组成的，每个规则返回的类型必须相同。

    模式匹配中使用_作为通配符。

    > //
    let isTrue c =
       match c with
       | true -> printfn "it's true"
       | false -> printfn "it's false";;

    val isTrue : bool -> unit

    > isTrue true;;
    it's true
    val it : unit = ()

    > //
    let num2str x=
       match x with
       | 1 -> "1"
       | 2-> "2"
       | _-> "不在范围内";;

    val num2str : int -> string

    > num2str 1;;
    val it : string = "1"
    > num2str 5;;
    val it : string = "不在范围内"

    匹配失败

    如果未找到匹配的规则，将抛出异常Microsoft.FSharp.Core.MatchFailureException。如果不想抛出异常，就必须处理所有的情况，如果F#编译器发现没有处理所有的情况，会给出一个警告。

    > //
    let num2str x=
       match x with
       | 1 -> "1"
       | 2 -> "2";;

    num2str 1;;
    ---------^

    stdin(174,10): warning FS0025: Incomplete pattern matches on this expression. For example, the value '0' may indicate a case not covered by the pattern(s).

    val num2str : int -> string

    > num2str 3;;
    Microsoft.FSharp.Core.MatchFailureException: The match cases were incomplete
       at FSI_0071.num2str(Int32 x)
       at <StartupCode$FSI_0072>.$FSI_0072.main@()
    Stopped due to error

    命名匹配(Named Patterns)

    可以绑定数据到新的值

    > //
    let rec fib x =
       match x with
       | 1 -> 1
       | 2 -> 1
       | y -> fib(x-2) + fib(x-1);;

    val fib : int -> int

    > fib 3;;
    val it : int = 2
    > fib 4;;
    val it : int = 3

    字面值模式(Literal Pattern)

    可以使用常量进行模式匹配，变量必须有[<Literal>]属性并且首字母必须大写。

    > //
    [<Literal>]
    let One = 1
    let isOne x =
       match x with
       | One -> "true"
       | _ -> "false";;

    val One : int = 1
    val isOne : int -> string

    > isOne 1;;
    val it : string = "true"
    > isOne 2;;
    val it : string = "false"

    when条件匹配

    > //
    let rec fib x =
       match x with
       | _ when x<=2 -> 1
       | _ -> fib(x-2) + fib(x-1);;

    val fib : int -> int

    > fib 3;;
    val it : int = 2

    分组模式

    > //
    let rec fib x =
       match x with
       | 1 | 2 -> 1
       | _ -> fib(x-2) + fib(x-1);;

    val fib : int -> int

    > fib 1;;
    val it : int = 1
    > fib 2;;
    val it : int = 1

    匹配结构化数据

    元组

    > let match_tuple t =
       match t with
       | 4,5 -> "匹配到(4,5)"
       | _,9 -> "匹配到(_,9)"
       | x,y -> sprintf "匹配到(%d,%d)" x y;;

    val match_tuple : int * int –> string

    列表

    > //
    let rec listlen list=
       match list with
       | [] -> 0
       | [_] -> 1
       | [_;_] -> 2
       | hd::rest -> 1 + listlen rest;;

    val listlen : 'a list -> int

    > listlen [1..5];;
    val it : int = 5

    可空类型

    > //
    let getValue o=
       match o with
       | Some(5) -> 5
       | Some(x) -> x
       | None -> 0;;

    val getValue : int option –> int

    在函数参数中的模式匹配

    > let add (Some(x)) (Some(y)) = x + y;;

         | Some(5) -> 5
    -------------------^^^^^^^

    stdin(244,20): warning FS0025: Incomplete pattern matches on this expression. For example, the value 'None' may indicate a case not covered by the pattern(s).

         | Some(5) -> 5
    ---------^^^^^^^

    stdin(244,10): warning FS0025: Incomplete pattern matches on this expression. For example, the value 'None' may indicate a case not covered by the pattern(s).

    val add : int option -> int option –> int

    > add (Some(4)) (Some(5));;
    val it : int = 9

---
从我的百度空间导入