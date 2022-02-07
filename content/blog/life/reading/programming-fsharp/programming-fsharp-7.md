+++
title = "《Programming F#》读书笔记七"
date=2010-03-09

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
异步

    async 表达式

    Async<’T>类型：

    Async.Start 异步执行

    Async.Parallel seq<Async<’T>> 并行执行

    Async.RunSynchronously同步执行，并获取返回值，通常在Async.Parallel之后调用

    异步中的异常处理：

    未捕捉的异常将中断进程

    1.可以在async表达式中使用try with表达式

    2.使用Async.Catch延迟处理异常

    例如：

    open System.IO
    let filePath = ""
    let processBytes data:byte[] =
        data

    async {
        printfn "Processing file [%s]" (Path.GetFileName(filePath))
        use fileStream = new FileStream(filePath, FileMode.Open)
        let bytesToRead = int fileStream.Length
        let! data = fileStream.AsyncRead(bytesToRead)
        printfn
            "Opened [%s], read [%d] bytes"
            (Path.GetFileName(filePath))
            data.Length
        let data' = processBytes data
        use resultFile = new FileStream(filePath + ".results", FileMode.Create)
        do! resultFile.AsyncWrite(data', 0, data'.Length)
        printfn "Finished processing file [%s]" <| Path.GetFileName(filePath)
    }
    |> Async.Catch
    |> Async.RunSynchronously
    |> function
    | Choice1Of2 result -> printfn "%A" result
    | Choice2Of2 (ex:exn) -> printfn "%s" ex.Message

    3.Async.StartWithContinuations

    Async.StartWithContinuations(
        superAwesomeAsyncTask,
        (fun result -> printfn "Task completed with result %A" result),
        (fun exn    -> printfn "Task threw an exception with Message: %s" exn.Message),
        (fun oce    -> printfn "Task was cancelled. Message: %s" oce.Message)
    )

    异步中的取消：

    不同于Thread.Abort()，调用取消不会立即终止线程，它会在下一次调用let!、do!等操作时停止执行剩余代码，执行取消处理代码。CancelDefaultToken用来跟踪是否执行取消处理,Async.CancelDefaultToken()取消最后一个启动的任务

    例：取消最后一个启动的任务

    open System
    open System.Threading
    let cancelableTask =
        async {
            printfn "Waiting 10 seconds..."
            for i = 1 to 10 do
                printfn "%d..." i
                do! Async.Sleep(1000)
            printfn "Finished!"
        }
    // Callback used when the operation is canceled
    let cancelHandler (ex : OperationCanceledException) =
        printfn "The task has been canceled."
    Async.TryCancelled(cancelableTask, cancelHandler)
    |> Async.Start
    // ...
    Async.CancelDefaultToken()

    如何取消所有异步任务：

    open System.Threading
    let computation = Async.TryCancelled(cancelableTask, cancelHandler)
    let cancellationSource = new CancellationTokenSource()
    Async.Start(computation, cancellationSource.Token)
    // ...
    cancellationSource.Cancel()

    Async.Sleep:

    Async.Sleep不同于Thread.Sleep，它不阻塞工作线程，sleep的线程被其他工作线程重用，当sleep结束，将唤醒另一个线程来完成剩余工作

    自定义异步扩展方法:

    使用Async.FromBeginEnd方法来自定义异步扩展方法

    open System
    open System.IO
    type System.IO.Directory with
        /// Retrieve all files under a path asynchronously
        static member AsyncGetFiles(path : string, searchPattern : string) =
            let dele = new Func<string * string, string[]>(Directory.GetFiles)
            Async.FromBeginEnd(
                (path, searchPattern),
                dele.BeginInvoke,
                dele.EndInvoke)

---
从我的百度空间导入