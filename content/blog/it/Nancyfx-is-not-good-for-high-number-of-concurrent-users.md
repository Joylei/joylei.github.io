+++
title="Nancyfx is not good for high number of concurrent users"
date=2016-08-01

[taxonomies]
categories=["Programming"]
tags=["Nancyfx", "HttpHandler", "Asp.Net"]
+++

Recently I was working on a application update service to support application self-update.
I choosed Nancyfx to implement the API service.

For the file downloading, we set up a target to support at least 5000 concurrent
 connections to download files at the same time with 2 cloud web servers behide load banlancer.

## Testing environment
Web-server
CPU: Quad 2.6 GHz AMD Opteron 6238
Memory: 16GB

Testing sample file: 5.5MB

## How it works
At first, to make things work, I used the AsFile() extension method of NancyRequest.
As you can imagine, the performance was poor, with high CPU and memory utilization.

To reduce the memory usage, I cached the file content in the memory since there
is small amount of files and no large files.
To impove the CPU usage and support large amount of concurrent connections, I used
 the async/await pattern to write file content directly the HttpResponse of Asp.Net.

To my surprise the momery usage was still crazy when performaning testing.

After several times of tries, I had a doubt that the issue was coming from Nancyfx.
As a comparision, I wrote a HttpHandler that handles the downloading directly.

With this handler, it can support more than 10k concurrent downloadings with 2 Web-servers,
 and the CPU utilization was about 60%~70% and memory usage was low. It seemed
 the bottleneck was the network bandwidth.
