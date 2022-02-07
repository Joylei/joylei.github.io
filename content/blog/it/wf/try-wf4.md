+++
title = "[WF4.0]Windows Workflow 4.0初体验"
date=2009-06-03

[taxonomies]
categories=["Programming"]
tags=["workflow", ".Net"]
+++
最近在虚拟机中安装了Visual Studio 2010。界面是WPF的，CPU和内存占用不是很夸张,打开一个很简单的附带的Lab Project, CPU使用率一般在20%一下，内存使用不到800M。

言归正传，还是来介绍Windows Workflow 4.0。

与3.5相比工作流模型有了很大改变和不同。

我们知道3.5中工作流都是托管在WorkflowRuntime中的，通过WorkflowRuntime来创建、执行工作流实例；在4.0中没有WorkflowRuntime类，可以方便的直接创建WorkflowInstance实例和执行工作流。Lab中代码如下：
```c#
WorkflowInstance myInstance = new WorkflowInstance(new SayHello(),
    new SayHelloInArgs(userName));
myInstance.OnCompleted = delegate(WorkflowCompletedEventArgs e)
{
    Console.WriteLine("*** OnCompleted delegate is running on thread {0} ***",
        Thread.CurrentThread.ManagedThreadId);
    SayHelloOutArgs outArgs = new SayHelloOutArgs(e.Outputs);
    greeting = outArgs.Greeting;
    syncEvent.Set();
};
myInstance.OnUnhandledException = delegate(WorkflowUnhandledExceptionEventArgs e)
{
    Console.WriteLine(e.UnhandledException.ToString());
    return UnhandledExceptionAction.Terminate;
};
myInstance.OnAborted = delegate(WorkflowAbortedEventArgs e)
{
    Console.WriteLine(e.Reason);
    syncEvent.Set();
};
myInstance.Run();

```
        
4.0中有一个WorkflowInvoker类，这个类也可以执行工作流，只不过这个类是用来测试工作流的，这很大的改进了前一版本中工作流难以测试的问题。
```c#
        [TestMethod]
        public void ShouldReturnGreetingWithName()
        {
            Dictionary<string, object> input = new Dictionary<string, object>()
            {
                {"UserName", "Test"}
            };
            IDictionary<string, object> output;
            output = WorkflowInvoker.Invoke(new SayHello(), input);
            Assert.AreEqual("Hello, Test from Workflow 4", output["Greeting"]);
        }
```
    
    
3.5中Activity是所有活动的基类，要实现自定义活动，只需重写Activity 的Execute()方法；在4.0中所有的活动都是从抽象类WorkflowElement派生出来的，而且Visual Studio中默认自定义活动都是从CodeActivity或CodeActivity<T>继承的，相似的是也需要重写Execute()方法，从而实现自定义执行逻辑。
```c#
    public class MyActivity1 : CodeActivity
    {
        protected override void Execute(CodeActivityContext context)
        {
            //你的实现代码
        }
    }
```
当然，你还是可以从Activity派生自定义活动，不过与3.5有很大不同。
```c#
    public class SayHelloInCode : Activity
    {
        protected override WorkflowElement CreateBody()
        {
            return new Sequence()
            {
                Activities =
                {
                    new WriteLine()
                    {
                        Text = "Hello Workflow 4 in code"
                    }
                }
            };
        }
    }
```


4.0中新增加的工作流服务功能，可以直接把工作流发布为WCF服务，当然工作流也必须设计为具有WCF应答功能才行。4.0提供4个与WCF相关的活动：Receive、ReceiveReply、Send、SendReply，通过这些活动可以可视化定义WCF的服务操作。
4.0中实现了工作流设计器的基本模型，可以很容易的实现自定义设计器。

---
从我的百度空间导入