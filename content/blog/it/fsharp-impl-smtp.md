+++
title = "F#实现简单的SMTP服务器"
date=2010-07-12

[taxonomies]
categories=["Programming"]
tags=["F#", "SMTP"]
+++
服务端代码
```f#
///SmtpLib.fs
namespace SmtpLib
    open System
    open System.Net
    open System.Net.Sockets
    open System.Text.RegularExpressions
    open System.IO
    open System.Threading
 
    type BufferManager (count:int) =
        let bufferStore:byte[] = Array.create (BufferManager.BufferSize*count) (byte 0)
        let used = new System.Collections.Generic.Queue()
        let storeOffset = ref 0
        let locker = new obj()
        static member BufferSize = 2048
        member r.SetBuffer (e:SocketAsyncEventArgs) = 
            lock (locker) (fun _->
                match storeOffset.Value with
                | value when value < Array.length(bufferStore)-1 -> 
                    e.SetBuffer(bufferStore,value,0)
                    storeOffset := storeOffset.Value + BufferManager.BufferSize
                | _ -> failwith "overflow"
            )
            
        member r.Push (e:SocketAsyncEventArgs) =
            lock (locker) (fun _->used.Enqueue e)|>ignore
        member r.Pop() =
            lock (locker) (fun _-> used.Dequeue())
 
    type SmtpCommand = |Wel=0 | Helo=1 | Auth=2 | Login1=3| Login2=4 | Mail=5 | Rcpt=6 | Data=7 | Quit=8
 
    type SmtpUserToken()=
        let mutable cmd = SmtpCommand.Wel
        let mutable buffer :ArraySegment option= None
        let mutable fromAddress:System.Net.Mail.MailAddress = null
        let toAddresses = new System.Net.Mail.MailAddressCollection()
        let mutable user = null
        let mutable pass = null
        let mutable domain = null
        let mutable isAuthed = false
 
        member r.LastCommand with get() = cmd
                             and set value = cmd <- value
 
        member r.Buffer with get() = buffer
                        and set value = buffer <- value
 
        member r.FromAddress with get() = fromAddress
                             and set value = fromAddress <- value
 
        member r.ToAddresses with get() = toAddresses
 
        member r.UserName with get() = user
                          and set value = user <- value
        member r.Password with get() = pass
                          and set value = pass <- value
        member r.Domain with get() = domain
                        and set value = domain <- value
        member r.IsAuthenticated with get() = isAuthed
                                 and set value = isAuthed <- value
        member r.Rset() =
            cmd <- SmtpCommand.Auth
            buffer <- None
            fromAddress <- null
            toAddresses.Clear()
        member r.Clear() = 
            cmd <- SmtpCommand.Wel
            buffer <- None
            fromAddress <- null
            toAddresses.Clear()
            user <- null
            pass <- null
            domain <- null
            isAuthed <- false
 
    ///RFC2821
    type SmtpServer(endPoint:IPEndPoint,maxConnections) =
        let _server = new Socket(endPoint.AddressFamily,SocketType.Stream,ProtocolType.Tcp)
        let _bufferManager = new BufferManager(maxConnections)
        let _maxClients = new Semaphore(maxConnections,maxConnections)
        new (maxConnection) = 
            new SmtpServer(new IPEndPoint(IPAddress.Loopback,25),maxConnection)
        new (ipAddress:IPAddress,port:int,maxConnections) =
            new SmtpServer(new IPEndPoint(ipAddress,port),maxConnections)
 
        ///初始化
        member r.Init() =
            printfn "%s initializing..." <|DateTime.Now.ToString()
            for i in [1..maxConnections] do
                let e = new SocketAsyncEventArgs()
                e.UserToken <- (new SmtpUserToken()) :>obj
                e.Completed.Add r.IO_Completed
                _bufferManager.SetBuffer e
                _bufferManager.Push(e)
 
        ///启动服务
        member r.RunForever()=
            r.Init()
 
            _server.Bind endPoint
            _server.Listen 1000
            let e = new SocketAsyncEventArgs()
            e.Completed.Add r.IO_Completed
            r.StartAccept e
            printfn "%s running at %s..." <|DateTime.Now.ToString() <|endPoint.ToString()
 
        member r.Stop()=
            if _server <> null then
                _server.Close()
 
        member r.StartAccept e =
            e.AcceptSocket <- null
            let pending = _server.AcceptAsync e
            if not pending then
                r.AcceptScoket e
 
        ///IO完成
        member r.IO_Completed e =
            match e.LastOperation with
            | SocketAsyncOperation.Accept -> r.AcceptScoket e
            | SocketAsyncOperation.Send -> r.AfterSend e
            | SocketAsyncOperation.Receive -> r.AfterReceive e
            | _ -> failwith "op not send,receive or accept"
 
        ///accept a socket
        member r.AcceptScoket e =
            if e.SocketError = SocketError.Success then
                //store local
                let client = e.AcceptSocket
                let arg = _bufferManager.Pop()
                arg.AcceptSocket <- client
                let token = arg.UserToken :?> SmtpUserToken
                printfn "%s %s connected" <|DateTime.Now.ToString() <|client.RemoteEndPoint.ToString()
                
                //reject connection 554
                //service down 421
                //welcome msg
                let msg = "220 fsharpsmtp welcome\r\n";
                r.StartSend arg msg
 
                _maxClients.WaitOne() |> ignore
                //next accept
                r.StartAccept e
 
        ///after send message
        member r.AfterSend e =
            let client = e.AcceptSocket
            let token = e.UserToken :?> SmtpUserToken
            let buffer = token.Buffer.Value
            let left = buffer.Count //剩余
            match e.SocketError,left,token.LastCommand with
            | SocketError.Success,0,SmtpCommand.Quit ->
                r.CloseSocket e
            | SocketError.Success,0,_ ->
                match token.LastCommand with
                | SmtpCommand.Quit ->
                    r.CloseSocket e //退出
                | _ ->
                    //初始数据存放缓冲
                    let bytes = Array.create BufferManager.BufferSize (byte 0)
                    let buffer = new ArraySegment(bytes,0,0)
                    token.Buffer <- Some(buffer)
                    let pending = client.ReceiveAsync e
                    if not pending then
                        r.AfterReceive e
            | SocketError.Success,_,_ ->
                let count = Array.min [|left;BufferManager.BufferSize|]//要发送数量
                let left = left - count
                let seg = new ArraySegment(buffer.Array,buffer.Offset + count,left)
                token.Buffer <- Some(seg)
                Buffer.BlockCopy(buffer.Array,buffer.Offset,e.Buffer,e.Offset,count)
                e.SetBuffer(e.Offset,count)
                let pending = client.SendAsync e
                if not pending then
                    r.AfterSend e
            | _ -> r.CloseSocket e
 
        //after receive message
        member r.AfterReceive e =
            let client = e.AcceptSocket
            let token = e.UserToken :?> SmtpUserToken
 
            match e.SocketError,e.BytesTransferred with
            | SocketError.Success,0 ->
                r.CloseSocket e
            | SocketError.Success,_ ->
                //可能接收到控制字符，强制关闭连接
                let index = Array.FindIndex(e.Buffer,e.Offset,e.BytesTransferred,(fun n-> n=byte 4))
                match index with
                | -1 -> 
                    let buffer = token.Buffer.Value
                    let left = buffer.Array.Length - buffer.Offset - 1 //缓冲剩余
                    if e.BytesTransferred > left then
                        let maxVal = Array.max [|e.BytesTransferred;buffer.Array.Length|]
                        Array.Resize(ref buffer.Array,maxVal*3)
 
                    Buffer.BlockCopy(e.Buffer,e.Offset,buffer.Array,buffer.Offset,e.BytesTransferred)
                    let buffer = new ArraySegment(buffer.Array,buffer.Offset+e.BytesTransferred,0)
                    token.Buffer <- Some(buffer)
                    match token.LastCommand,buffer.Offset>=5,buffer.Offset>=2 with
                    | SmtpCommand.Data,true,_ -> //接收数据
                        //TODO:552 Too much mail data
                        let endOfData = [|byte 13;byte 10;byte 46;byte 13;byte 10|]
                        let lastData = Array.sub buffer.Array (buffer.Offset-5) 5
                        let arr = Array.zip endOfData lastData
                        let isEnd = Array.exists (fun (x,y)->x=y|>not) arr  |>not
                        if isEnd then
                            //重置命令状态
                            token.Rset()
                            //接收数据结束,保存到文件
                            let queueId = Guid.NewGuid();
                            let cwd = Directory.GetCurrentDirectory()
                            let fileName = sprintf "%s\\%s.dat" cwd <|queueId.ToString()
                            async {
                                use! fs = File.AsyncOpen(fileName,FileMode.Create,FileAccess.Write,FileShare.Write,4096*1024,FileOptions.Asynchronous)
                                do! fs.AsyncWrite(buffer.Array,0,buffer.Offset - 5)
                                printfn "%s data saved to file:%s" <|DateTime.Now.ToString() <|fileName
                                let msg = "250 Message accepted\r\n"
                                r.StartSend e msg
                            }|> Async.Start
                            
                        else
                            let pending = client.ReceiveAsync e
                            if not pending then
                                r.AfterReceive e
                    | SmtpCommand.Data,false,_ ->
                        let pending = client.ReceiveAsync e
                        if not pending then
                            r.AfterReceive e
                    | _,_,true ->
                        let endOfData = [|byte 13;byte 10|]
                        let lastData = Array.sub buffer.Array (buffer.Offset-2) 2
                        let arr = Array.zip endOfData lastData
                        let isEnd = Array.exists (fun (x,y)->x=y|>not) arr  |>not
                        if isEnd then
                            //接收数据结束
                            let input = System.Text.Encoding.ASCII.GetString(buffer.Array,0,buffer.Offset-2)
                            //auth login命令用户名和密码分开接收
                            match token.LastCommand with
                            | SmtpCommand.Login1 ->
                                let msg = 
                                    try
                                        let bytes = Convert.FromBase64String(input)
                                        token.UserName <- System.Text.Encoding.ASCII.GetString(bytes)
                                        token.LastCommand <- SmtpCommand.Login2
                                        "334 UGFzc3dvcmQ6\r\n"
                                    with
                                    | _ -> 
                                        "501 invalid character\r\n"
                                r.StartSend e msg
                            | SmtpCommand.Login2 ->
                                let msg = 
                                    try
                                        let bytes = Convert.FromBase64String(input)
                                        token.Password <- System.Text.Encoding.ASCII.GetString(bytes)
                                        if r.CheckUser token.UserName token.Password then
                                            token.IsAuthenticated <- true
                                            token.LastCommand <- SmtpCommand.Auth
                                            "235 OK Authenticated\r\n"
                                        else
                                            "535 Authenticated Failed\r\n"
                                    with
                                    | _ -> 
                                        "501 invalid character\r\n"
                                r.StartSend e msg
                            | _ -> r.ParseCmd e (input.ToLower())
                        else
                            let pending = client.ReceiveAsync e
                            if not pending then
                                r.AfterReceive e
 
                    | _,_,_ -> //接收消息
                        let pending = client.ReceiveAsync e
                        if not pending then
                            r.AfterReceive e
                | _ -> r.CloseSocket e
            | SocketError.TimedOut,_ ->
                let msg = "421 connection timeout,closing transmission channel\r\n"
                token.LastCommand <- SmtpCommand.Quit
                r.StartSend e msg
            | _,_ -> r.CloseSocket e
        member r.CheckUser user pass =
            //TODO:check user & pass
            true
 
        member r.ParseCmd e input=
            let client = e.AcceptSocket
            printfn "%s %s:%s"  <|DateTime.Now.ToString() <| client.RemoteEndPoint.ToString() <|input
            let token = e.UserToken :?> SmtpUserToken
            let  msg():string =
                //The maximum total length of a command line including the command
                //word and the  is 512 characters
                if input.Length = 0 || input.Length>510 then
                    "500 Syntax error, command unrecognized\r\n"
                else
                    let words = input.Split([|' '|])
                    if words.Length = 0 then
                        "500 invalid args\r\n"
                    else
                        let cmd = words.[0]
                        let args = 
                            if words.Length = 0 then
                                [||]
                            else
                                Array.sub words 1 <| words.Length - 1
                        match cmd with
                        | "helo" ->
                            r.ProccessHelo token args
                        | "ehlo" ->
                            r.ProccessEhlo token args
                        | "auth" ->
                            r.ProccessAuth token args
                        | "mail" ->
                            r.ProccessMail token args
                        | "rcpt" ->
                            r.ProccessRcpt token args
                        | "data" ->
                            r.ProccessData token args
                        | "quit" ->
                            r.ProccessQuit token args
                        | "noop" ->
                            "250 ok\r\n"
                        | "rset" ->
                            r.ProccessRset token args
                        | _ -> "500 Syntax error, command unrecognized\r\n"
                        //502 Command not implemented
            msg()
            |> r.StartSend e 
        ///start session
        member r.ProccessHelo token args=
            if args.Length =1 && not (args.[0].Length=0) then
                //TODO:verify domain
                token.Clear() //clear session
                //TODO:addition operation like ip authentication
                token.IsAuthenticated <- true
                token.Domain <- args.[0]
                token.LastCommand <- SmtpCommand.Helo
                "250 ok\r\n"
            else
                "501 Syntax error in parameters or arguments\r\n"
        ///start session
        member r.ProccessEhlo token args=
            if args.Length =1 && not (args.[0].Length=0) then
                token.Clear() //clear session
                //TODO:addition operation like ip authentication
                token.IsAuthenticated <- true
                token.Domain <- args.[0]
                token.LastCommand <- SmtpCommand.Helo
                "250-ok\r\n250 AUTH LOGIN PLAIN\r\n"
            else
                "501 Syntax error in parameters or arguments\r\n"
 
        member r.ProccessAuth token args=
            match token.LastCommand with
            | SmtpCommand.Helo
            | SmtpCommand.Auth
            | SmtpCommand.Login1
            | SmtpCommand.Login2  ->
                if args.Length > 0 then
                    match args.Length,args.[0] with
                    | 1,"login" ->
                        token.LastCommand <- SmtpCommand.Login1
                        "334 VXNlcm5hbWU6\r\n"
                    | 2,"login" | 3,"plain" ->
                        try
                            let bytes = System.Convert.FromBase64String(args.[2])
                            let namepass = System.Text.Encoding.ASCII.GetString(bytes)
                            let parts = namepass.Split([|'\000'|])
                            token.UserName <- parts.[0]
                            token.Password <- parts.[1]
                            if r.CheckUser token.UserName token.Password then
                                token.IsAuthenticated <- true
                                token.LastCommand <- SmtpCommand.Auth
                                "235 OK Authenticated\r\n"
                            else
                                "535 Authentication Failed\r\n"
                        with
                        | _ -> "501 Syntax error\r\n"
                    | _,_ -> "501 Syntax error in parameters or arguments\r\n"
                else
                    "501 Syntax error in parameters or arguments\r\n"
            | _ -> "503 bad sequence of commands\r\n"
 
        /// MAIL FROM: [SP  ] .
        /// This command tells the SMTP-receiver that a new mail transaction is
        /// starting and to reset all its state tables and buffers.
        /// If accepted, the SMTP server returns a 250 OK reply;
        /// failures produce 550 or 553 replies;
        /// start transaction
        member r.ProccessMail token args=
            let msgAddr addr =
                match token.IsAuthenticated with
                | false ->
                    "535 Authentication Required\r\n"
                | true ->
                    try
                        token.Rset() //reset state
                        //TODO:user limit 64,domain limit 255
                        let address = new System.Net.Mail.MailAddress(addr);
                        token.FromAddress <- address
                        token.LastCommand <- SmtpCommand.Mail
                        //TODO:550 policy reject
                        //251 User not local; will forward to 
                        sprintf "250 %s Accepted\r\n" address.Address
                    with
                    | _ as ex ->
                        "501 mail from should like \"mail from:\"\r\n"
 
            match token.LastCommand with
            | SmtpCommand.Helo | SmtpCommand.Auth | SmtpCommand.Login2 ->
                if args.Length = 1 then
                    let reg = new Regex("from:<([^>]+@[^>]+)>")
                    let m = reg.Match(args.[0])
                    if m.Success then
                        msgAddr m.Groups.[1].Value
                    else
                        "501 Syntax error in parameters or arguments\r\n"
                else if args.Length=2 && args.[0] = "from:" then
                    let reg = new Regex(@"<([^>]+@[^>]+)>")
                    let m = reg.Match(args.[0])
                    if m.Success then
                        msgAddr m.Groups.[1].Value
                    else
                        "501 Syntax error in parameters or arguments\r\n"
                else
                    "501 Syntax error in parameters or arguments\r\n"
            | _ -> "503 bad sequence of commands\r\n"
 
        /// RCPT TO: [ SP  ] .
        /// If accepted, the SMTP server returns a 250 OK.
        /// If the recipient is known not to be a deliverable address,
        /// the SMTP server returns a 550 reply,
        /// typically with a string such as "no such user - " and the mailbox name
        member r.ProccessRcpt token args=
            let msgAddr addr =
                try
                    let address = new System.Net.Mail.MailAddress(addr);
                    token.ToAddresses.Add(address)
                    token.LastCommand <- SmtpCommand.Rcpt
                    //TODO:452 Too many recipients
                    //TODO:550 policy reject
                    sprintf "250 %s Accepted\r\n" address.Address
                with
                | _ as ex ->
                    "501 rcpt to should like \"rcpt to:\"\r\n"
 
            match token.LastCommand with
            | SmtpCommand.Mail | SmtpCommand.Rcpt ->
                if args.Length = 1 then
                    let reg = new Regex("to:<([^>]+@[^>]+)>")
                    let m = reg.Match(args.[0])
                    if m.Success then
                        msgAddr m.Groups.[1].Value
                    else
                        "501 Syntax error in parameters or arguments\r\n"
                else if args.Length=2 && args.[0]="to:" then
                    let reg = new Regex("<([^>]+@[^>]+)>")
                    let m = reg.Match(args.[1])
                    if m.Success then
                        msgAddr m.Groups.[1].Value
                    else
                        "501 Syntax error in parameters or arguments\r\n"
                else
                    "501 Syntax error in parameters or arguments\r\n"
            | _ -> "503 bad sequence of commands\r\n"
 
        /// If there was no MAIL, or no RCPT, command, or all such commands
        /// were rejected, the server MAY return a "command out of sequence"
        /// (503) or "no valid recipients" (554) reply in response to the DATA
        /// command
        member r.ProccessData token args =
            match args.Length>0,token.LastCommand with
            | true,_-> "500 invalid syntax"
            | _,SmtpCommand.Rcpt ->
                token.LastCommand <- SmtpCommand.Data
                "354 Enter mail, end with \".\"\r\n"
            | _,_ -> "503 bad sequence of commands\r\n"
 
        member r.ProccessQuit token args =
            match args.Length with
            | 0 ->
                token.LastCommand <- SmtpCommand.Quit
                "221 bye\r\n"
            | _ -> "501 Syntax error in parameters or arguments"
 
        member r.ProccessRset token args = 
            match args.Length with
            | 0 -> 
                token.Rset()
                "250 ok\r\n"
            | _ -> "501 Syntax error in parameters or arguments"
 
        member r.StartSend e msg=
            let client = e.AcceptSocket
            let token = e.UserToken :?> SmtpUserToken
            let bytes = System.Text.Encoding.ASCII.GetBytes(msg)
            let count = Array.min [|bytes.Length;BufferManager.BufferSize|]
            let left = bytes.Length - count
 
            let buffer = new ArraySegment(bytes,count,left)
            token.Buffer <- Some(buffer)
            e.SetBuffer(e.Offset,count)
            Buffer.BlockCopy(buffer.Array,0,e.Buffer,e.Offset,count)
 
            let pending = client.SendAsync e
            if not pending then
                r.AfterSend e
 
        ///关闭连接
        member r.CloseSocket e=
            let client = e.AcceptSocket
            printfn "%s %s disconnected" <|DateTime.Now.ToString() <|client.RemoteEndPoint.ToString()
            client.Shutdown SocketShutdown.Both
            client.Close()
            let token = e.UserToken :?> SmtpUserToken
            token.Clear()
            e.AcceptSocket <- null
         
///Program.fs
module Main=
    open SmtpLib
    open System
    []
    let main(args:string[]) =
        let s = new SmtpServer(1000)
        s.RunForever()
        Console.ReadLine()|>ignore
        0
```

客户端代码
```f#
///Program.fs
open System.Net.Mail
open System.ComponentModel
[]
let main (args:string[]) =
    let client = new SmtpClient("127.0.0.1",25)
    let msg = new MailMessage("someone@mailfrom.com","someone@rcptto.com")
    msg.Subject <- "subject test"
    msg.Body <- @"mail body:
    line2
    line3
    line4"
    try
        //client.Send(msg)
        client.SendCompleted.Add (fun _->printfn "success")
        client.SendAsync(msg,null)
                
    with
    | ex ->
        printfn "%s" <|ex.ToString()
    System.Console.ReadKey() |> ignore
    0

```
---
从我的百度空间导入