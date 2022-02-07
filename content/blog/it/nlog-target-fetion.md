+++
title = "编写NLog飞信扩展"
date=2010-03-24

[taxonomies]
categories=["Programming"]
tags=[".Net", "c#", "NLog"]
+++

 `NLog`是个功能很丰富的日志记录工具，我们可以很方便的扩展日志目标。当系统发生严重错误时，我们希望通过短信得到通知，用飞信`API(fetionlib)`写一个飞信扩展，就得到了免费的短信提醒。
 
 ## 编写NLog扩展:

很简单，只需从`TargetWithLayout`继承，重写`Write`方法就行了。

FetionTarget.cs
```c#

    using System;
    using System.Collections.Generic;
    using System.Linq;
    using System.Net;
    using System.Net.Cache;
    using System.Text;
    using Newtonsoft.Json;
    using System.IO;
    using System.Text.RegularExpressions;
    using System.Threading;
    using System.Web;

    using NLog;

    namespace FetionLib
    {
        [Target("Fetion")]
        public class FetionTarget : TargetWithLayout
        {
            public FetionTarget()
            {
            }

            /// <summary>
            /// 基本地址
            /// </summary>
            public string BaseAddress { get; set; }

            /// <summary>
            /// 登录手机号
            /// </summary>
            public string Mobile { get; set; }
            /// <summary>
            /// 用户密码
            /// </summary>
            public string Password { get; set; }

            /// <summary>
            /// 收信人
            /// </summary>
            public string Recipients { get; set; }

            /// <summary>
            /// 超时时限
            /// </summary>
            public string Timeout { get; set; }

            /// <summary>
            /// 缓存消息数量
            /// </summary>
            public int BufferCount { get; set; }

            protected override void Write(LogEventInfo logEvent)
            {
                string message = CompiledLayout.GetFormattedMessage(logEvent);

                Send(message);
            }

            protected override void Write(LogEventInfo[] logEvents)
            {
                foreach (var eventInfo in logEvents)
                {
                    Write(eventInfo);
                }
            }

            /// <summary>
            /// 发送消息
            /// </summary>
            /// <param name="message"></param>
            /// <returns></returns>
            private void Send(string message)
            {
                if(string.IsNullOrEmpty(BaseAddress))
                {
                    Console.WriteLine("BaseAddress为空");
                    return;
                }

                if (string.IsNullOrEmpty(Mobile))
                {
                    Console.WriteLine("Mobile为空");
                    return;
                }

                if (!Regex.IsMatch(Mobile, @"\d+"))
                {
                    Console.WriteLine("Mobile格式不正确");
                    return;
                }

                List<string> friends = new List<string>();
                friends.Add(Mobile);//发送到自己

                if (!string.IsNullOrEmpty(Recipients))
                {
                    var parts = Recipients.Split(new char[] { ';' }, StringSplitOptions.RemoveEmptyEntries);

                    if (parts.Length != 0)
                    {
                        var items = (from fetionId in parts
                                     select fetionId.Trim()).ToArray();
                        friends.AddRange(items);
                    }
                }

                Send(message, friends.Distinct().ToArray());
            }

            /// <summary>
            /// 群发
            /// </summary>
            /// <param name="message"></param>
            /// <param name="friends"></param>
            /// <returns></returns>
            private void Send(string message, string[] friends)
            {
                int timeOut = 0;
                if (!int.TryParse(Timeout, out timeOut) || timeOut <= 0)
                {
                    timeOut = 3000;
                }

                string uuid = Guid.NewGuid().ToString();

                JsonSerializer serializer = new JsonSerializer();
                StringWriter swriter = new StringWriter();
                serializer.Serialize(swriter, friends);
                string friendUri = swriter.ToString();

                string content = "mobile=" + Mobile + "&uuid=" + uuid
                                 + "&password=" + Password + "&friend=" + friendUri
                                 + "&message=" + HttpUtility.UrlEncode(message, Encoding.UTF8);
                var request = HttpWebRequest.Create(string.Format("{0}/restlet/fetion", BaseAddress));
                request.Method = "POST";
                request.CachePolicy = new RequestCachePolicy(RequestCacheLevel.NoCacheNoStore);
                request.ContentType = "application/x-www-form-urlencoded";
                request.Timeout = timeOut;
                var buffer = Encoding.ASCII.GetBytes(content);
                int bufferSize = 1024;
                int offset = 0;
                int count = buffer.Length > bufferSize ? 1024 : buffer.Length;
                AsyncCallback responseCallback = delegate(IAsyncResult ar)
                {
                    var response = (HttpWebResponse)request.EndGetResponse(ar);
                    if (response.StatusCode == HttpStatusCode.Accepted)
                    {
                        //success
                        Console.WriteLine("发送成功");
                    }
                    else
                    {
                        //fail
                        Console.WriteLine("发送失败");
                    }
                };
                AsyncCallback writeCallBack = null;
                writeCallBack = delegate(IAsyncResult ar)
                {
                    var requestStream = (Stream)ar.AsyncState;
                    try
                    {
                        requestStream.EndWrite(ar);
                        offset += count;
                        int left = buffer.Length - offset;
                        if (left > 0)
                        {
                            count = left > bufferSize ? bufferSize : left;
                            requestStream.BeginWrite(buffer, offset, count,
                                                     writeCallBack,
                                                     requestStream);
                        }
                        else
                        {
                            request.BeginGetResponse(responseCallback, null);
                        }
                    }
                    catch (Exception ex)
                    {
                        //fail
                        Console.WriteLine("fail:{0}",ex.Message);
                    }

                };
                AsyncCallback requestCallBack = delegate(IAsyncResult ar)
                {

                    var requestStream = request.EndGetRequestStream(ar);
                    requestStream.BeginWrite(buffer, offset, count, writeCallBack,
                                             requestStream);

                };
                request.BeginGetRequestStream(requestCallBack, null);

            }
        }
    }
```

编译项目得到FetionLib.dll。

通过配置文件使用：
```xml

    <?xml version="1.0"?>
    <configuration>
    <configSections>
        <section name="nlog" type="NLog.Config.ConfigSectionHandler, NLog"/>
    </configSections>
    <nlog xmlns="http://www.nlog-project.org/schemas/NLog.xsd" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">
        <extensions>
          <add assembly="FetionLib"/>
        </extensions>
        <targets>
          <!--默认收信人为发信人，recipients收信人用;分隔多个-->
          <target name="fetion" type="Fetion"
                  BaseAddress="http://fetion.xinghuo.org.ru"
                  mobile="135*******" password="******"
                  layout="${date:format=HH\:MM\:ss} ${message}"/>
          <target name="console" xsi:type="Console" layout="${date:format=HH\:MM\:ss} ${logger} ${message}"/>
        </targets>
        <rules>
          <logger name="*" minLevel="Info" appendTo="fetion"/>
          <logger name="*" minLevel="Info" appendTo="console"/>
        </rules>
    </nlog>
    <startup><supportedRuntime version="v4.0" sku=".NETFramework,Version=v4.0"/></startup>
    </configuration>
```

在控制台测试：

```c#
    using System;
    using NLog;

    namespace NLogTest
    {
        class Program
        {
            static void Main(string[] args)
            {
                Logger logger = LogManager.GetCurrentClassLogger();
                logger.Error("something is wrong!");
                Console.ReadKey();
            }
        }
    }
```

启动运行后看到"发送成功"你就收到短信了。

## 相关资源：

[1.How To Write Your Own Target](http://nlog-project.org/how-to-write-your-own-target.html)

[2.飞信接口API调用举例](http://fetionlib.appspot.com/api.html)

---
从我的百度空间导入