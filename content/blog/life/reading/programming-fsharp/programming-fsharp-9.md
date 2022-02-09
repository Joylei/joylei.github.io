+++
title = "《Programming F#》读书笔记九"
date=2010-06-27

[taxonomies]
categories=["Reading"]
tags=["Programming", "F#", ".Net"]
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
List.reduce与List.fold

    List.reduce:以list的第一个元素作为初始值，要求list中至少有一个元素，否则抛出异常System.ArgumentException。聚合的结果的类型必须与list中的元素的类型相同。

    如：

    > let acc r n = r + n;;

    val acc : int -> int -> int

    > List.reduce acc [];;
    System.ArgumentException: The input list was empty.
    Parameter name: list
       at Microsoft.FSharp.Collections.ListModule.Reduce[T](FSharpFunc`2 reduction, FSharpList`1 list)
       at <StartupCode$FSI_0006>.$FSI_0006.main@()
    Stopped due to error
    > List.reduce acc [1];;
    val it : int = 1
    > List.reduce acc [1..5];;
    val it : int = 15

    > let acc (r:string) n = r + n.ToString();;

    val acc : string -> 'a -> string

    > List.reduce acc [1..5];;

    List.reduce acc [1..5];;
    ----------------^^^^^^

    stdin(11,17): error FS0001: The type 'string' does not support any operators named 'get_One'

    List.fold:需要指定初始值，聚合结果的类型不必与list中的元素的类型相同，list也可以为空集合。

    > List.fold acc "" [];;
    val it : string = ""
    > List.fold acc "" [1..5];;
    val it : string = "12345"

函数编程语言关键特征：

    不可变数据
    组合使用函数
    函数可用作数据
    延迟计算
    模式匹配

匿名函数

    C#中叫lambda表达式

    let add = fun x y-> x+y

    add 3 4

部分函数

    > let add x y = x + y;;

    val add : int -> int -> int

    > let part = add 3;;

    val part : (int -> int)

    > let c = part 4;;

    val c : int = 7

    在C#中进行类似的操作：
                 Func<int,int,int> add = (x,y)=>x+y;
                 Func<int,int> part = y=>add(3,y);
                 int c = part(4);
                 Console.WriteLine(c);如果只有一个参数，可以使用匿名部分函数来避免使用lambda表达式：

    > List.iter (printfn "item is %d\r\n") [1..5];;
    item is 1

    item is 2

    item is 3

    item is 4

    item is 5

    val it : unit = ()

Pipe-Forward Operator（|>）

    可以把调整书写顺序，把最后一个参数写在前面，使得代码看起来更顺畅和简单。

    > let compute dir =
        dir
        |>Directory.GetFiles
        |>Array.map (fun file-> new FileInfo(file))
        |>Array.map (fun info->info.Length)
        |>Array.sum;;

    val compute : string -> int64

    > compute @"D:\";;
    val it : int64 = 2220474411L

    > let computeOrdered dir=
        Array.sum (
            Array.map (fun (info:FileInfo)->info.Length)
                (Array.map (fun file->new FileInfo(file))
                    (Directory.GetFiles dir)
                )
        );;

    val computeOrdered : string -> int64

    > computeOrdered @"D:\";;
    val it : int64 = 2220474411L

Forward composition operator（>>）

    把嵌套调用的函数组合成一个从左到右的顺序调用的函数，它不需要在定义时指定参数，它的参数是第一个函数的参数：let f g x = f(g(x))

    > let computeComposed =
        Directory.GetFiles
        >>Array.map (fun file-> new FileInfo(file))
        >>Array.map (fun info->info.Length)
        >>Array.sum;;

    val computeComposed : (string -> int64)

    > computeComposed @"D:\";;
    val it : int64 = 2220474411L

    Pipe-backward operator（<|）

    用于调整运算的优先级，使<|右边部分先计算，可以省掉一对括号

    > add 5 (add 1 2);;
    val it : int = 8
    > add 5 <| add 1 2;;
    val it : int = 8

Backward composition operator(<<)

    把嵌套调用的函数组合成一个从右到左的顺序调用的函数，与Forward composition operator（>>）的作用相反；let f g x = g(f(x))

    > let computeBC =
        Array.sum
        <<Array.map (fun (info:FileInfo)->info.Length)
        <<Array.map (fun file-> new FileInfo(file))
        <<Directory.GetFiles;;

    val computeBC : (string -> int64)

    > computeBC @"D:\";;
    val it : int64 = 2220474411L

    可以看出这个顺序和上文中computeOrdered函数中的调用顺序是一致的。

---
从我的百度空间导入