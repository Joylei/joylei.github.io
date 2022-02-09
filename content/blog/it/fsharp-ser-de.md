+++
title = "F#序列化与反序列化"
date=2010-09-10

[taxonomies]
categories=["Programming"]
tags=["F#", "serialization", "xml", ".Net"]
+++
借助F#熟悉一下序列化与反序列化。
```f#
namespace PersonType
open System
open System.IO
open System.Xml.Serialization
open System.Runtime.Serialization.Formatters.Binary[<Serializable>]
type Person= val mutable 
    private name:string val mutable 
    private birthday:DateTime val mutable 
    private gender:int 
    new ()={name=null;birthday=DateTime.Now;gender=0} 
    member public r.Name with get()=r.name and set v=r.name<-v 
    member public r.Birthday with get()=r.birthday and set v=r.birthday<-v 
    member public r.Gender with get() = r.gender and set v=r.gender<-v
module Program= 
    [<EntryPoint>] 
    let main(args)=
        let p = new Person(Name="test",Birthday=DateTime.Now,Gender=1)
        printfn "%A" p //xml序列化 let xml = new XmlSerializer(typeof<Person>); 
        use writer = new StreamWriter(@"D:\fs\Person.xml") xml.Serialize(writer,p) 
        writer.Close() 
        use reader = new StreamReader(@"D:\fs\Person.xml") let p1 =xml.Deserialize(reader) :?> Person 
        printfn "%s,%A,%A" p1.Name p1.Birthday p1.Gender //二进制序列化 
        let formatter = new BinaryFormatter(); 
        use writer = File.OpenWrite(@"D:\fs\Person.dat") formatter.Serialize(writer,p); 
        writer.Close() 
        use reader = File.OpenRead(@"D:\fs\Person.dat"); 
        let p2 = formatter.Deserialize(reader) :?> Person 
        printfn "%s,%A,%A" p2.Name p2.Birthday p2.Gender 
        Console.Read() |> ignore 0
```

对于xml序列化来说，不需要类型有Serializable属性。

此外：发现在F#中模块中的类型不能进行二进制序列化操作，所以在上面的程序中我把它放在一个命名空间下。

---
从我的百度空间导入