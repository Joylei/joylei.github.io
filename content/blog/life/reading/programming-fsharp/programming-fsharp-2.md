+++
title = "《Programming F#》读书笔记二"
date=2010-01-23


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
Discriminated Unions 可区分的联合类型

    与C中的union很像，可以存储多种类型的数据，但是同时只能一种类型起作用。实际上它的每个case都是一个类，能用枚举的最好用枚举

    type Color = Red | Green | White   //定义了一个不能关联数据的联合

    type Flower =

           | Rose of Color

           | Orchid of Color

           | PaperFlower of string //定义了一个可以关联数据的联合

    let redRose = Rose(Red)
    let whiteOrchid = Orchid(White)
    let paperRose = PaperFlower("Rose")

    用于模式匹配：

    let kindOfFlower =
    function
    | Rose(_) -> "Rose"
    | Orchid(_) -> "Orchid"
    | PaperFlower(_) -> "PaperFlower"

    联合还可以有方法和属性，下面为Flower定义一个属性:

    type Flower =

           | Rose of Color

           | Orchid of Color

           | PaperFlower of string

           member this.Value =
    match this with
    | Rose(_) -> "Rose"
    | Orchid(_) -> "Orchid"
    | PaperFlower(_) -> "PaperFlower"

Record类型

    type Person = {Name:string;Age:int}

    隐式的创建一个Person对象

    let p1 = {Name=”Bob”;Age = 20}

    如果同时有多个定义相同的Record类型，就必须显示的指定为哪一个

    let p2:Person = {Name=”Bob”;Age=20}

    复制一个Record对象，这里必须使用with

    let p3 = {p2 with Age=22}

    用于模式匹配：

    let filter =
    function
    | {Age=20} -> true //匹配Age为20的Person对象
    | _ -> false

    Record类型也可以有方法和属性

           member this.Value = sprintf “{Name:%s;Age:%d}” this.Name this.Age

Lazy延迟类型：

    使用函数创建

    let x = Lazy<int>.Create(fun() ->
    let a = p2.Age - 10
    printfn "a=%d" a
    a
    )

    使用表达式创建

    let y = lazy(x.Value + 10)

    当调用Value时才进行计算求值

Seq序列类型：

    let rangs = seq { 1..5 } //创建一个Seq

    Seq是.Net中的IEnumerable<T>的别名；与List类型的区别是Seq是延迟创建的，等到用时才按照定义创建迭代项;而List则在定义时就生成所有数据项。

    let rec allFiles dir =      //递归遍历目录及子目录下所有文件

          seq { yield! Directory.GetFiles(dir)             //seq是一个Computation Expression;yield!在seq中合并子序列的每一项到父序列

               for subDir in Directory.GetDirectories(dir) do

                     yield! allFiles subDir

         }

默认值：

    使用Unchecked.defaultof<T> 来获取某种类型的默认值

    引用类型未初始化时默认为null，引用类型不能赋值为null

    type Animal =
    val mutable name :string
    new () =
    {name = "default"}
    new (n) ={name = n}
    member this.Name = this.name

    let mutable a = new Animal()
    a <- new Animal("a2")

    a <- null //这里会报错

mutable可变类型：

    let x = 2

    x <- 3 //会报错，不是可变类型

    let mutable x = 2

    x <- 3 //修改x值为3

    使用mutable类型有限制：

    mutable类型数据只可以在定义它的函数范围内使用，不包括内部函数

    let invalidUseOfMutable() =
        let mutable x = 0
        //let incrementX() = x <- x + 1 //如果去掉注释会报错
        //incrementX()
        x

    mutable record:

    type Person = {Name:string;mutable Age:int}

    let p = {Name=”Sam”;Age = 20}

    p.Age <- 30 //修改值

ref类型：

    ref关键字将原先分配在stack上的值分配到heap上

    let x = 2

    let y = ref x //通过变量来创建ref变量

    y:=3 //使用:=操作符来修改ref变量的值，修改y的值并不会改变x的值

    let z = ref 3 //通过字面值来创建ref类型


---
从我的百度空间导入