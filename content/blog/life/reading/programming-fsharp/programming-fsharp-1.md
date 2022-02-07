+++
title = "《Programming F#》读书笔记一"
date=2010-01-23


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
注释：

       //单行注释

      (*

            多行注释

       *)

       ///用于Visual Studio中文档注释

Unit类型

       Unit类型只有一个值就是空元组()，可以认为是F#中的void类型。

       F#函数必须有返回值，当没有数据要返回时就默认返回unit。

Option类型

       Option类型是可空类型。

       let y = None //空值

       let y= Some 20

       Option.get //获取值

       Option.isSome y //判断是否有值

       Option.isNone y //判断是否为空

       用于模式匹配：

       let myMatch param =

             match param with

             | Some(20) -> printf "匹配到20"

             | Some(_) -> printf "匹配到值"

             | None -> printf "为空"

function关键字：

        用于只带一个参数的模式匹配，简化语句

       如上面的myMatch函数可以简化为

       let myMatch =

             function

             | Some(20) -> printf "匹配到20"

             | Some(_) -> printf "匹配到值"

             | None -> printf "为空"

Tuple元组：

          let x = 2,3,4 //用逗号隔开，不加括号

          let y = (4,5) //加括号

          当元组中有且仅有两个元素时可以用fst和snd函数来获取元组中的第一个和第二个元素。

          其他情况下用类似下面的方式来获取元组中的值

          let x,y = 2,3       //x = 2,y = 3

          let x,_ = 4,5       //使用通配符，隐藏不想要的值

List列表：

         let x = [2;3;4]

          let x = [ for i in 2..5 -> i*5 ] //动态产生List

         let x = [ for i in 2..5 do

                          yield i*5]          //动态产生List

         let x = 30::x      //使用::把一个元素连接到List的头部

         用于模式匹配：

         let myMatch =
              function
              | [] -> printf "匹配空List"
               //| [x] -> printf "匹配含有一个元素的List"
              | [x;y] -> printf "匹配含有两个元素的List"
              | x::[] -> printf "匹配含有一个元素的List"
              | _ -> printf "其他情况"

自定义操作符：

       操作符可以有以下字符“!%&*+-./<=>?@^|~”；还有“:”，它不能用在操作符第一个位置。

        有两个参数的操作符默认是中缀操作符，~!?开头的操作符是前缀操作符。

       let (@@) x = x*x

       let (@@@) x y = x + y //两个参数的操作符是中缀操作符

       let (~@@@@) x y = x * y    //~!?开头的操作符是前缀操作符

       可以在高阶函数中直接使用操作符，只需在操作符两边加上括号。

        List.reduce (@@@) [1..5]

        List.reduce (+) [1..5]


---
从我的百度空间导入