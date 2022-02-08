+++
title = "《Programming F#》读书笔记十一"
date=2010-07-18

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
### 引用表达式(quotation expression)


引用表达式代表F#抽象语法树
- <@ @>代表泛型的Expr
- <@@ @@>代表非泛型的Expr
表达式匹配


### 常量值
- Int32(i)匹配整型
- Double(f)匹配浮点型
- String(s)匹配字符串
- 成员方法或静态方法
- Call(obj,method,args)匹配所有的成员方法
- lambda表达式
Lambda(var,args)匹配lambda表达式
- SpecificCall
SpecificCall用来匹配具体的某一个方法或函数

```f#
#r "FSharp.PowerPack.dll"
open System
open Microsoft.FSharp.Quotations
open Microsoft.FSharp.Quotations.Patterns
open Microsoft.FSharp.Quotations.DerivedPatterns

let rec printExpr(expr:Expr)=
   match expr with
   | Int32(i) -> printfn "整型:%d" i
   | Double(f) -> printfn "浮点型:%f" f
   | String(s) -> printfn "字符串:%s" s
   | SpecificCall <@ (+) @> (_,_,[lft;rgt]) ->
      printfn "SpecificCall:%A+%A" lft rgt
   | Call(o,meth,args) ->
      let c =
         match o with
         | Some(x) -> sprintf "%O" x
         | None -> "(static method)"
      printfn "方法%s.%s(%A)" c (meth.Name) args
   | Lambda(var,body)->
      printfn "lambda value:%s type:%s" var.Name var.Type.Name
      printfn "lambda body:"
      printExpr(body)
   | _ -> printfn "unwanted expr:%A" expr

printExpr <@ 1+1 @>
printExpr <@ System.Console.ReadLine() @>
let add(x,y) = x+y
printExpr <@ add @>
Console.ReadLine()
```
 
### 函数体引用表达式


F#默认不能返回函数或成员方法函数体的引用表达式。
函数加了[`<ReflectedDefinition>`]属性之后，F#会把函数体的定义持久化到一个表中，嵌入到dll或exe中，这样就可以获取到函数体的引用表达式。
```f#
open System
open Microsoft.FSharp.Quotations
open Microsoft.FSharp.Quotations.Patterns
open Microsoft.FSharp.Quotations.DerivedPatterns

//对比注释掉下一行和未注释的输出
[<ReflectedDefinition>]
let add(x,y) = x+y

let task1()=
   let rec printExpr(expr:Expr)=
   match expr with
   | Int32(i) -> printfn "整型:%d" i
   | Double(f) -> printfn "浮点型:%f" f
   | String(s) -> printfn "字符串:%s" s
   | SpecificCall <@ (+) @> (_,_,[lft;rgt]) ->
      printfn "SpecificCall:%A+%A" lft rgt
   | Call(o,meth,args) ->
      let c =
         match o with
         | Some(x) -> sprintf "%O" x
         | None -> "(static method)"
      printfn "方法:%s.%s(%A)" c (meth.Name) args
      match meth with
      |MethodWithReflectedDefinition(m) ->
         printfn "展开%s" meth.Name
         printExpr(m)
      | _ -> printfn "不能展开：%s" meth.Name
   | Lambda(var,body)->
      printfn "lambda value:%s type:%s" var.Name var.Type.Name
      printfn "lambda body:"
      printExpr(body)
   | _ -> printfn "unwanted expr:%A" expr

printExpr <@ add(1,2) @>
Console.ReadLine() |>ignore

[<EntryPoint>]
let main args =
   task1()
   0
```
 
### 通用匹配

可以用来匹配大部分有意义的Expr，减少工作量。可以和前面的其它匹配一起使用。

- SharpVar(var) 匹配值
- ShapeLambda(var,body)匹配函数
- ShapeCombination(_,exprs)匹配复杂的表达式，分解出所有子表达式

```f# 
#r "FSharp.PowerPack.dll"
open Microsoft.FSharp.Quotations
open Microsoft.FSharp.Quotations.ExprShape

let rec printExpr(expr:Expr)=
    match expr with
    | ShapeVar(var) -> printfn "var:%s" var.Name
    | ShapeLambda(var,body) ->
        printfn "lambda:%s" var.Name
        printfn "lambda body:"
        printExpr body
    | ShapeCombination(_,exprs) ->
        printfn "combination:"
        exprs
        |> List.iter printExpr

let add a b = a + b
printExpr <@ add @>
```

### 创建表达式
```
> let expr =
> <@ let x = (1,2)
> (x,x)@>
> ;;

val expr : Expr<(int * int) * (int * int)> =
Let (x, NewTuple (Value (1), Value (2)), NewTuple (x, x))

> let expr =
>Expr.Let(new Var("x",typeof<int*int>),
>Expr.NewTuple([Expr.Value(1);Expr.Value(2)]),
>Expr.NewTuple([Expr.GlobalVar("x");Expr.GlobalVar("x")]));;

val expr : Expr = Let (x, NewTuple (Value (1), Value (2)), NewTuple (x, x))
```
 
### Expression Hole

使用符号在引用表达式中表示一个引用表达式。

%在泛型表达式中使用，%%在非泛型表达式中使用

```
> let add a b = <@ %a + %b @>;;
val add : Expr<int> -> Expr<int> -> Expr<int>
> add <@ 1 @> <@ 2 @>;;
val it : Expr<int> =
Call (None, Int32 op_Addition[Int32,Int32,Int32](Int32, Int32),
[Value (1), Value (2)])
```
 

### 表达式计算
```f#
#r "System.Core.dll"
#r "FSharp.PowerPack.dll"
#r "FSharp.PowerPack.Linq.dll"
open Microsoft.FSharp.Linq.QuotationEvaluation;;
let expr = <@ fun (a,b) -> a+b @>
let func = expr.Compile()();
let func2 = expr.Eval()
```

```
> val expr : Expr<(int * int -> int)> =
> Lambda (tupledArg,
> Let (a, TupleGet (tupledArg, 0),
> Let (b, TupleGet (tupledArg, 1),
> Call (None, Int32 op_Addition[Int32,Int32,Int32](Int32, Int32),
> [a, b]))))
val func : (int * int -> int)
val func2 : (int * int -> int)

> func2(1,2);;
val it : int = 3

> func(1,2);;
val it : int = 3
```

---
从我的百度空间导入