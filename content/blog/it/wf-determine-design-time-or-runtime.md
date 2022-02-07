+++
title = "[WF3.5]如何判断活动在设计时还是在运行时"
date=2009-03-25

[taxonomies]
categories=["Programming"]
tags=["workflow", ".Net"]
+++

Rehosting工作流设计器,有时需要判断活动是在运行时还是在设计时，比如说对活动进行验证时，可以通过获取自定义服务来判断。

以下是一个自定义ActivityValidator：
```c#
public class ActionValidator : ActivityValidator
    {
        public override ValidationErrorCollection Validate(ValidationManager manager, object obj)
        {
            .......
           
            MarkupOnlyWorkflow root = ActivityHelper.GetRootActivity(action) as MarkupOnlyWorkflow;
            TemplateWrapper wrapper = (TemplateWrapper)manager.GetService(typeof(TemplateWrapper));
            WorkflowTemplate template = null;
            if (wrapper == null)//运行时
            {
                template = EntityUtilities.GetInstance().GetWorkflowTemplate(root.TemplateId);
            }
            else//设计时
            {
                template = wrapper.WorkflowTemplate;
            }
            //进行相关操作
            .......
        }
    }
```
---
从我的百度空间导入