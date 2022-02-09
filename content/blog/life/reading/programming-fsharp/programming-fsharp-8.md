+++
title = "《Programming F#》读书笔记八"
date=2010-03-09

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

并行框架

Parallel.For

Array.Parallel

Array.Parallel.iter

Array.parallel.map

Array.Parallel.choose

.Net并行扩展 (PFX)

    PFX按照CPU剩余资源和CPU处理器个数决定同时处理多少个任务

    let task = Task.Factory.StartNew<string>(taskBody) //创建一个并行任务

    let x = task.Result

    访问Result将进行以下处理：

    1.如果任务已结束，则返回结果

    2.如果任务已开始，等待任务完成

    3.如果任务未执行，则在当前线程执行

Task.WaitAll 等待所有任务完成

Task.WaitAny 等待任一任务完成
取消任务:

    需要自行判断何时取消任务

    open System
    open System.Threading
    open System.Threading.Tasks
    let longTaskCTS = new CancellationTokenSource()
    let longRunningTask() =
        let mutable i = 1
        let mutable loop = true
        while i <= 10 && loop do
            printfn "%d..." i
            i <- i + 1
            Thread.Sleep(1000)
            // Check if the task was cancelled
            if longTaskCTS.IsCancellationRequested then
                printfn "Cancelled; stopping early."
                loop <- false
        printfn "Complete!"
    let startLongRunningTask() =
        Task.Factory.StartNew(longRunningTask, longTaskCTS.Token)
    let t = startLongRunningTask()
    // ...   

    longTaskCTS.Cancel()

异常：

    任务抛出System.AggregateException类型包装的异常

    try
        let treasureFindingTask =

            Task.Factory.StartNew<int>(fun () -> findAllTreasure colossalCave)
        printfn "Found %d treasure!" treasureFindingTask.Result
    with
    | :? AggregateException as ae ->
        // Get all inner exceptions from the aggregate exception.
        let rec decomposeAggregEx (ae : AggregateException) =
            seq {
                for innerEx in ae.InnerExceptions do
                    match innerEx with
                    | :? AggregateException as ae -> yield! decomposeAggregEx ae
                    | _ -> yield innerEx
            }
        printfn "AggregateException:"
        // Alternately, you can use ae.Flatten() to put all nested
        // exceptions into a single AggregateException collection.
        decomposeAggregEx ae
        |> Seq.iter (fun ex -> printfn "\tMessage: %s" ex.Message)


---
从我的百度空间导入