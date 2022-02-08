+++
title = "《Programming F#》读书笔记五"
date=2010-01-27

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

### 接口

定义接口：
```f#
type IPerson =   //所有方法和属性都是抽象的
        abstract Name:string
        abstract Age:int
        abstract TalkWith:IPerson –> unit

type IPerson = //另一种形式
        interface
            abstract Name:string
            abstract Age:int
            abstract TalkWith:IPerson –> unit

        end
```

实现接口：

F#中实现接口必须实现接口中的所有属性和方法；实现接口时必须实现接口的所有基接口中的所有属性和方法。
```f#
type IPerson =   //所有方法和属性都是抽象的
        abstract Name:string with get,set //可读写属性
        abstract Age:int //只读属性
        abstract TalkWith:IPerson -> unit //方法

type IStudent =
        inherit IPerson
        abstract ClassRoom:string

type Student =
    val mutable name:string
    val mutable age:int
    val mutable room:string
    new() = {name="";age=0;room="room1"}
    interface IPerson with
        override this.Name with get() = this.name
                            and set value = this.name <- value
        override this.Age with get() = this.age
                            //and set value = this.age <- value
        override this.TalkWith(p:IPerson) =
            printfn "talk with %s" p.Name
    interface IStudent with
        override this.ClassRoom with get() = this.room
                                //and set value = this.room <- value
```

调用接口

    F#中对象必须显示的转换成接口才能调用接口成员

```f#
let s = new Student()
let i = s :> IPerson
let n = i.Name
```
   
###  对象表达式

匿名对象,与java中的类似，可以直接实现接口和抽象类
```f#
let o = {
    new IComparer<IPerson> with
        member this.Compare(l,r) =
            if l.Age > r.Age then 1
            elif lAge = r.Age then 0
            else -1
}
```

### 扩展方法
```
type System.Int32 with
    member this.ToHexString() = sprintf "0x%x" this;;
```

扩展方法必须定义在模块中，只能在F#中起作用，.net不能识别

### 扩展模块

    只要名字相同就就是扩展
```f#
namespace FSCollectionExtensions
open System.Collections.Generic
module List =
    /// Skips the first n elements of the list
    let rec skip n list =
        match n, list with
        | _, []       -> []
        | 0, list     -> list
        | n, hd :: tl -> skip (n - 1) tl
```

### 枚举
```f#
type Color =
        | Red = 0
        | Green= 1
        | Black= 2

let i:int = int Color.Green //枚举值转换为int
let o = enum<Color>(i) //int转换为枚举值
```

### 结构体
```f#
[<Struct>]
type StructPoint(x : int, y : int) =
    member this.X = x
    member this.Y = y
type StructRect(top : int, bottom : int, left : int, right : in
    struct
        member this.Top    = top
        member this.Bottom = bottom
        member this.Left   = left
        member this.Right = right
        override this.ToString() =
            sprintf "[%d, %d, %d, %d]" top bottom left right
    end
```
不能使用隐式构造，不能重写默认构造；其它结构体同C#中的结构体表现一样。

### Active Pattern

Single-Case Active Patterns:
```f#
open System.IO

let (|FileExtension|) filePath = Path.GetExtension(filePath) //定义了一个active pattern
let determineFileType (filePath : string) =
    match filePath with
    // Without active patterns
    | filePath when Path.GetExtension(filePath) = ".txt"
        -> printfn "It is a text file."
    // Converting the data using an active pattern
    | FileExtension ".jpg"
    | FileExtension ".png"
    | FileExtension ".gif"
        -> printfn "It is an image file."
    // Binding a new value
    | FileExtension ext
        -> printfn "Unknown file extension [%s]" ext
```

Partial-Case Active Patterns: //只匹配返回结果为Some(_)的值
```f#
open System
let (|ToBool|_|) x =
    let success, result = Boolean.TryParse(x)
    if success then Some(result)
    else            None
let (|ToInt|_|) x =
    let success, result = Int32.TryParse(x)
    if success then Some(result)
    else            None

let describeString str =
    match str with
    | ToBool b -> printfn "%s is a bool with value %b" str b     
    | ToInt   i -> printfn "%s is an integer with value %d" str i
    | _         -> printfn "%s is not a bool, int, or float" str
```

Parameterized Active Patterns://参数紧跟在pattern名称之后
```f#
open System.Text.RegularExpressions
// Use a regular expression to capture three groups
let (|RegexMatch3|_|) (pattern : string) (input : string) =
    let result = Regex.Match(input, pattern)
    if result.Success then
        match (List.tail [ for g in result.Groups -> g.Value ]) with
        | fst :: snd :: trd :: []
                -> Some (fst, snd, trd)
        | [] -> failwith <| "Match succeeded, but no groups found.\n" +
                            "Use '(.*)' to capture groups"
        | _ -> failwith "Match succeeded, but did not find exactly three group"
    else
        None
let parseTime input =
    match input with
    // Match input of the form "6/20/2008"
    | RegexMatch3 "(\d+)/(\d+)/(\d\d\d\d)" (month, day, year)
    // Match input of the form "2004-12-8"
    | RegexMatch3 "(\d\d\d\d)-(\d+)-(\d+)" (year, month, day)
        -> Some( new DateTime(int year, int month, int day) )
    | _ –> None
```

Multi-Case Active Patterns:
```f#
let (| Hour | Minite | Second | Other |) (text:string) =
    let parts = text.Split(':')
    if Array.length parts = 3 then
        Hour(int parts.[0],int parts.[1],int parts.[2])
    elif Array.length parts = 2 then
        Minite(int parts.[0],int parts.[1])
    elif Array.length parts = 1 then
        Second(int parts.[0])
    else
        Other

let seconds =
    function
    | Hour (h,i,s) -> h*3600 + i * 60 + s
    | Minite (i,s) -> i * 60 + s
    | Second (s) -> s
    | _ -> 0
```

---
从我的百度空间导入