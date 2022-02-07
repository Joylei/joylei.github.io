+++
title = "解决外部网站提交Form表单乱码问题"
date=2009-11-22

[taxonomies]
categories=["Programming"]
tags=["c#", "asp.net", "encoding"]
+++
有时处理的FORM表单是乱码，这是因为提交页和处理时的编码不同造成的。

如果两个页面都是所用的编码都是可控制的，那就没什么好说的，配置或修改一致就可以了。

但是外部网站是不可控的，所以只能由本地网站上在接收时进行判断处理。

`Asp.net`的接收数据处理受配置文件中的`requestEncoding`的影响，默认使用utf-8编码。这样接收到得数据就使用这个编码来进行解码，所以如果能够判断出数据所用的编码，并在解码时使用这个新的编码，那就解决问题了。幸好`Asp.net`提供了读取原始输入流的方法和设置处理编码的属性。

以下是一个简单的示例：

使用`gb2312`编码的`test.html`

```html
<html>
<head>
<title>test</title>
<meta http-equiv="Content-Type" content="text/html; charset=GB2312" />
</head>
<body>
<form action="TestHandler.ashx" method="post" enctype="application/x-www-form-urlencoded">
<input name="txtName" id="txtName" type="text" value=""/>
<input type="submit" value="提交"/>
</from>
</body>
```

处理页面`TestHandler.ashx`
```c#
public class TestHandler : IHttpHandler {
   
    public void ProcessRequest (HttpContext context) {
        var rawBytes = context.Request.BinaryRead(context.Request.ContentLength);
        var decodedBytes = HttpUtility.UrlDecodeToBytes(rawBytes);
       
        //如果没有下面的判断，很可能乱码
        if(!IsUTF8Bytes(decodedBytes))
        {
            context.Request.ContentEncoding = Encoding.GetEncoding("GB18030");
        }
       
        context.Response.ContentType = "text/plain";
        context.Response.Write(context.Request.Form["txtName"]);
    }

    //网上找的一个简单判断utf-8编码方法

    public bool IsUTF8Bytes(byte[] data)
    {


        int charByteCounter = 1;　 //计算当前正分析的字符应还有的字节数

        byte curByte; //当前分析的字节.

        for (int i = 0; i < data.Length; i++)
        {
            curByte = data[i];

            if (charByteCounter == 1)
            {

                if (curByte >= 0x80)
                {
                    //判断当前
                    while (((curByte <<= 1) & 0x80) != 0)
                    {

                        charByteCounter++;
                    }

                    //标记位首位若为非0 则至少以2个1开始 如:110XXXXX...........1111110X　
                    if (charByteCounter == 1 || charByteCounter > 6)
                    {
                        return false;
                    }

                }
            }
            else
            {
                //若是UTF-8 此时第一位必须为1
                if ((curByte & 0xC0) != 0x80)
                {
                    return false;
                }
                charByteCounter--;
            }
        }


        if (charByteCounter > 1)
        {
            throw new Exception("非预期的byte格式");
        }

        return true;
    }
   
    public bool IsReusable {
        get {
            return false;
        }
    }

}
```

可以看出这里只是简单判断了是否`uft-8`编码，也许你需要更复杂的编码探测程序，这不是本文探讨的内容。

希望本文对有相同问题困扰的朋友有所帮助。

---
从我的百度空间导入