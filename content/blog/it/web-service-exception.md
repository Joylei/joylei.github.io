+++
title = "关于web服务服务端的异常"
date=2008-04-16

[taxonomies]
categories=["Programming"]
tags=["c#", "webservice"]
+++

在调试web服务时，系统内置抛出的异常会在网页上显示出来；用户抛出的异常系统不处理，响应为http500错误，被包装成了SoapException。

---
从我的百度空间导入