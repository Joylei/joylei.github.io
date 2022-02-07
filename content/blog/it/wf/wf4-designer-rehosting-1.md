+++
title = "[WF4.0]工作流设计器Rehosting（一）"
date=2009-06-27

[taxonomies]
categories=["Programming"]
tags=["workflow", ".Net"]
+++
## 索引
- [[WF4.0]工作流设计器Rehosting（一）](@/blog/it/wf/wf4-designer-rehosting-1.md)
- [[WF4.0]工作流设计器Rehosting（二）](@/blog/it/wf/wf4-designer-rehosting-2.md)
- [[WF4.0]工作流设计器Rehosting（三）](@/blog/it/wf/wf4-designer-rehosting-3.md)


## 正文
因为WF4.0使用WPF做可视化设计，能够利用WPF的数据绑定和其他一些内部实现，极大的简化了工作流设计器的开发工作；不用像3.5和3.0中那样，要开发一些服务类来支持工作流的设计操作。

首先看看WorkflowDesigner类的相关信息,WorkflowDesigner提供一个设计画布来呈现工作流模型。

System.Activities.Design.WorkflowDesigner

相关方法和属性

public object Deserializestring(string text)

从xaml工作流反序列化工作流对象

public void Flush()

保存工作流设计内容到Text属性

public bool IsInErrorState（）

是否处于错误状态

public void Load(object instance)

从工作流根活动对象加载到设计器

public void Load(string fileName)

从Xaml文件加载工作流到设计器

public void Save(string fileName)

保存为Xaml形式的工作流

public WorkflowDesigner()

构造函数

public System.Activities.Design.EditingContext Context{get;}

获取设计上下文对象，该对象包含一系列用于设计器和宿主交互的服务

public System.Windows.Controls.ContextMenu ContextMenu { get; }

设计器中活动的上下文菜单

public System.Activities.Design.Debug.IDesignerDebugView DebugManagerView { get; }

提供运行时调试服务对象

public string PropertyGridFontAndColorData { set; }

设置PropertyGrid字体和颜色

public System.Windows.UIElement PropertyInspectorView { get; }

提供PropertyGrid视图

public string Text { set; get; }

获取和设置Xaml工作流内容

public System.Windows.UIElement View { get; }

提供工作流可视化设计视图

public event System.Windows.Controls.TextChangedEventHandler TextChanged

当Text属性内容改变时触发

---
从我的百度空间导入