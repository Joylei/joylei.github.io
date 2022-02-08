+++
title = "《Programming F#》读书笔记十二"
date=2010-07-19

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
索引器：

```f#
open System
type Addresses(size:int)=
    let arr = ResizeArray<string>(seq{for _ in 0..size -> null})
    //索引器
    member r.Item with get idx = arr.[idx]
        and set idx value = arr.[idx] <- value

    member r.Print()=
        printfn "%A" arr

let t = Addresses(10)
t.[0] <- "abc"
t.Print()
t.[1] <- "def"
t.Print()
printf "t.[1] = %s" <|t.[1]
Console.ReadKey()
```
输出：
```
seq ["abc"; null; null; null; ...]
seq ["abc"; "def"; null; null; ...]
t.[1] = def
```
 
切片：

要实现切片只需要实现GetSlice方法，最多可以添加二维的取值范围

一维切片
```f#
type Addresses<'a>(items:seq<'a>)=
    let arr = ResizeArray<'a>(items)
    member r.GetSlice(downBound:int option,upBound:int option)=
        let downVal =
            match downBound with
            | Some(v) -> v
            | None -> 0
        let upVal =
            match upBound with
            | Some(v) -> v
            | None -> arr.Count
        let subItems = seq{ for i in downVal..upVal -> arr.[i]}
        Addresses(subItems)
    member r.Print()=
        printfn "Addresses:%A" arr

let nums = seq{for i in 0..10 -> sprintf "item%d" i};;
let addrs = Addresses nums
addrs.Print()
let subs = addrs.[1..4]
subs.Print()
```

二维切片
```f#
type Points(items:seq<int*int>)=
    member r.GetSlice(xDownBound:int option,xUpBound:int option,yDownBound:int option,yUpBound:int option)=
        let filter_x x =
        match xDownBound,xUpBound with
        | Some(down),Some(up) -> x >= down && x <= up
        | Some(down),None -> x >= down
        | None,Some(up) -> x <= up
        | None,None -> true
        let filter_y y =
        match yDownBound,yUpBound with
        | Some(down),Some(up) -> y >= down && y <= up
        | Some(down),None -> y >= down
        | None,Some(up) -> y <= up
        | None,None -> true
        let subItems = items |> Seq.filter (fun (x,y) -> filter_x(x) && filter_y(y))
        Points subItems
    static member Create()=
        let items = seq{
            for x in 0..10 do
                for y in 0..10 do
                    yield x,y}
        Points items
    member r.Print()=
        printfn "Points:%A" items
```

操作符重载
```f#
type Vector2DWithOperators(dx:float,dy:float) =
    member x.DX = dx
    member x.DY = dy
    static member (+) (v1: Vector2DWithOperators ,v2: Vector2DWithOperators) =
        Vector2DWithOperators(v1.DX + v2.DX, v1.DY + v2.DY)
    static member (-) (v1: Vector2DWithOperators ,v2: Vector2DWithOperators) =
        Vector2DWithOperators (v1.DX - v2.DX, v1.DY - v2.DY)
```
 

命名可选参数

命名可选参数只能用于类的构造函数

```f#
open System.Drawing
type LabelInfo(?text:string, ?font:Font) =
    let text = defaultArg text ""
    let font = match font with
    | None -> new Font(FontFamily.GenericSansSerif,12.0f)
    | Some v -> v
member x.Text = text
member x.Font = font
```
```
> LabelInfo (text="Hello World");;
val it : LabelInfo =
{Font = [Font: Name=Microsoft Sans Serif, Size=12]; Text = "Hello World"}
> LabelInfo("Goodbye Lenin");;
val it : LabelInfo =
{Font = [Font: Name=Microsoft Sans Serif, Size=12];  Text = "Goodbye Lenin"}
> LabelInfo(font=new Font(FontFamily.GenericMonospace,36.0f),
text="Imagine");;
val it : LabelInfo =
{Font = [Font: Name=Courier New, Size=36]; Text = "Imagine"}
```
 

属性初始化

用法和命名可选参数类似，都是在构造函数中设置
```f#
open System.Windows.Forms
let form = new Form(Visible=true,TopMost=true,Text="Welcome to F#")
```
方法重载
```f#
type Vector =
    { DX:float; DY:float }
    member v.Length = sqrt(v.DX*v.DX+v.DY*v.DY)
type Point =
    { X:float; Y:float }
    static member (-) (p1:Point,p2:Point) = { DX=p1.X-p2.X; DY=p1.Y-p2.Y }
    static member (-) (p:Point,v:Vector) = { X=p.X-v.DX; Y=p.Y-v.DY }
```
 
```
> let v = {DX=2.0;DY=3.0};;
val v : Vector = {DX = 2.0;DY = 3.0;}

> let p = {X=3.0;Y=4.0};;
val p : Point = {X = 3.0;Y = 4.0;}

> p-p;;
val it : Vector = {DX = 0.0;DY = 0.0;}

> p-v;;
val it : Point = {X = 1.0;Y = 1.0;}
```
---
从我的百度空间导入