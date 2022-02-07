+++
title = "Visual Studio 2010的一个Bug"
date=2009-06-24

[taxonomies]
categories=["Programming"]
tags=["visual studio", "bug"]
+++

装了VS2010和Sqlserver2008后，发现CPU使用率时不时跳到100%，检查不是杀毒软件的问题，发现msiexec.exe这个进程很活跃，查看事件日志，果然安装程序在不断地执行一个事务，详细事件报告如下：
```xml
<Event xmlns="http://schemas.microsoft.com/win/2004/08/events/event"> <System> <Provider Name="MsiInstaller" /> <EventID Qualifiers="0">1004</EventID> <Level>3</Level> <Task>0</Task> <Keywords>0x80000000000000</Keywords> <TimeCreated SystemTime="2009-06-23T10:48:42.000000000Z" /> <EventRecordID>18545</EventRecordID> <Channel>Application</Channel> <Computer>daysky-PC</Computer> <Security UserID="S-1-5-18" /> </System> <EventData> <Data>{316EE0C1-DB94-30BA-95E6-F4959035EE4B}</Data> <Data>VB_for_VS_7_Ent_28_x86_enu</Data> <Data>{F5686A3B-40B1-40CF-8B60-88D08F50A38A}</Data> <Data>D:\Win7\Visual Studio 2010\Common7\IDE\PrivateAssemblies\EN\</Data> <Data>(NULL)</Data> <Data>(NULL)</Data> <Data /> </EventData>

</Event>
<Event xmlns="http://schemas.microsoft.com/win/2004/08/events/event"> <System> <Provider Name="MsiInstaller" /> <EventID Qualifiers="0">1001</EventID> <Level>3</Level> <Task>0</Task> <Keywords>0x80000000000000</Keywords> <TimeCreated SystemTime="2009-06-23T10:48:42.000000000Z" /> <EventRecordID>18546</EventRecordID> <Channel>Application</Channel> <Computer>daysky-PC</Computer> <Security UserID="S-1-5-18" /> </System> <EventData> <Data>{316EE0C1-DB94-30BA-95E6-F4959035EE4B}</Data> <Data>VB_for_VS_7_Ent_28_x86_enu</Data> <Data>{97909034-45AB-48C6-853B-1CA8B3FBFB58}</Data> <Data>(NULL)</Data> <Data>(NULL)</Data> <Data>(NULL)</Data> <Data /> </EventData>

</Event>
```

通过搜索"F5686A3B-40B1-40CF-8B60-88D08F50A38A"，发现已经有兄弟把此问题提交到微软了，而且大概一个德国兄弟给出了解决方案，解决问题很简单，在"D:\Win7\Visual studio 2010\Common7\IDE\PublicAssemblies"目录下新建一个"en"目录，随即解决了这个问题。同时安装了Visual Studio 2008的话，也需要建立"en"目录。

---
从我的百度空间导入