+++
title = "《Programming F#》读书笔记四"
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
对象相等：

    元组、可区分的联合、Record都是引用类型，但是在相等比较时进行的是内容比较

类与继承:

    F#中编译器不会提供默认构造函数

    显示构造函数:

    type Person =

          val mutable name:string //声明字段，尚未初始化;不能在声明时初始化

          val mutable age:int

          static member InstanceCount = 0 //定义一个静态成员并初始化

          new () = {name = "myName"; age = 0}//初始化

          new(n,a) =

                printfn "初始化时可以做额外的操作"

                {name=n;age=a}//初始化

                then

                     printfn "初始化完成可以做额外的操作"

          member this.Name with get() = this.name      //有get和set方法的属性，必须通过this引用字段，get方法必须有括号
                                      and set value = this.name <-value //必须把and和with对齐

          member this.Age with get() = this.age
                                   and set value = this.age <- value

          member this.Text = sprintf "name:%s;age:%d" this.name this.age   //只有get方法的属性

          member this.TalkWith(sb:Person) =                   //类方法
                 printfn "I’m talking with %s" sb.Name

    隐式构造函数：

    type Person(n,a) =
        let mutable name = n //不能显式的定义字段，let定义的变量会在编译后转化为类字段
        let mutable age = a

        do
            printfn "初始化后进行的操作"

        static member InstanceCount = 0
        new() = new Person("",0)
        new (text:string) =
            let parts = text.Split(',')
            let nm = parts.[0]
            let ag = int parts.[1]
            new Person(nm, ag)

        member this.Age with get() = age //不是类成员，不能通过this引用
                                 and set value = age <- value

         类方法：

          类方法递归时不需要rec关键字

    泛型：

          type Person<’a>

          泛型参数以英文单引号开头

    在构造函数中初始化属性：
    let p = new Person(Name="Bob",Age=30)

    可访问性：public | private | internal

         默认可访问性是public //书上说字段默认为private，经验证字段默认也是public

         模块中的成员也可以使用可访问性

         可以使用fsi签名文件来控制可访问性，类似于C++中的头文件；不在签名文件中的代码默认为private，签名中的默认为public

    继承：

          从有显示构造函数的类继承：

    type Student=
        inherit Person
        //let mutable _sno = sno
        val mutable _sno:int
        new(text:string) =
            {
                inherit Person(text)
                _sno = 0
            }
        member this.SNO with get() = this._sno
                        and set value = this._sno <- value

    从有隐式构造函数的类继承;

    type Student(sno:int)=
        inherit Person() //必须调用基类构造函数
        let mutable _sno = sno
        member this.SNO with get() = _sno
                        and set value = _sno <- value

    重写：

          要重写的方法和属性使用abstract标记，可以使用default为属性提供默认实现

    type Sandwich() =
        abstract Ingredients : string list
        default this.Ingredients = []
        abstract Calories : int
        default this.Calories = 0

    [<AbstractClass>]属性标记抽象类

    [<Sealed>]属性标记Sealed类

    类型转换：

    let p = new Person()

    let o = p :> obj //静态转换

    let p2 = obj :?> Person //动态转换

    用于模式匹配

    let whatIs (x : obj) =
        match x with
        | :? string    as s -> printfn "x is a string \"%s\"" s
        | :? int       as i -> printfn "x is an int %d" i
        | :? list<int> as l -> printfn "x is an int list '%A'" l
        | _ -> printfn "x is a '%s'" <| x.GetType().Name;;


---
从我的百度空间导入