+++
title = "《Programming F#》读书笔记三"
date=2010-01-24

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
数组：

    let x = [| 1;1;3;5|]

    let x = [| 1..10 |]

    let x = [| for i in 1..10 –> i*i |]

    let y = x.[1]

    x.[1] <- 20

    let z = x.[0..3]

    let z = x.[6..]

    let z = x.[..6]

    let z = x.[*]

    let len = Array.length x

    let exists = Array.exists (fun item –> item>50) x         //判断数组是否满足给定条件

    Array.init 根据索引初始化数组

    let z = Array.init 10 (fun index –> index * 10)

    Array.zeroCreate 使用默认值初始化数组

    let z:int array = Array.zeroCreate 10          //初始化为0

    let z:string array = Array.zeroCreate 10      //初始化为null

    用于模式匹配：

    let myMatch =

         function

         | null –> printf “匹配到null”

         | [| |] –> printf “匹配到空数组”

         | [| x |] –> printf “匹配到一个元素的数组”

         | [| x;y |] –> printf “匹配到两个元素的数组”

         | _ –> printf “匹配到更多元素的数组”

    数组相等的判断

    F#中数组相等比较的是数组的内容，而.net中比较的数组的引用

    let eq = [| 0;1;2 |] = [| 0..2 |]   //相等

    let eq = [| 0;1;2 |] = [| 0..10 |] //不等

    规则数组

    let x:int [,] = Array2D.zeroCreate 3 3

    let y = x.[1,1]

    x.[1,1] <- 30

    let y = x.[*,1..2]

    不规则数组，数组的数组

    let x:int [][] = Array.zeroCreate 3;;

循环：

    while循环

    let mutable i= 10

    while i>0 do

          printf “%d;” i

          i <- i – 1

       for循环

    for i = 1 to 10 do

          printf “%d;” i

    for i = 10 downto 1 do

          printf “%d;” i

    for i in 1..10 do

           printf “%d;” i

    F#中没有break和continue，要达到类似的目的，必须使用while语句。

    for循环中进行模式匹配：

    > for a,b in [for x in 1..10 -> (x,x*x)] do printfn "%d" (a+b);;
    2
    6
    12
    20
    30
    42
    56
    72
    90
    110
    val it : unit = ()

异常：

    自定义F#中的异常

    exception WebSiteNotFound of string

    exception Timeout

    捕捉和抛出异常

    try

          raise WebSiteNotFound(“http://myWebsite.com”)

    with

    | :? FileNotFoundException

           -> printf “.Net FileNotFoundException”

    | :? IOException as ex

         -> printf “.Net IOException:%s” ex.Message

    | WebSiteNotFound(webSite)

         -> printf “F# WebSiteNotFound:%s” webSite

    | Timeout

        -> printf “F# Timeout”

    | _ as ex

        -> reraise ex   //重新抛出异常

    finally

          printf “finally”

---
从我的百度空间导入