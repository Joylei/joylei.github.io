+++
title = "Asp.Net扩展TreeView功能"
date=2010-01-06

[taxonomies]
categories=["Programming"]
tags=["c#", "asp.net", "treeview", "jquery"]
+++
```html
<script type="text/javascript" src="../JS/jquery.min.js"></script>
<script type="text/javascript">
$(function() {
    //禁止点击时 的postback
    $("#<%=tvProducts.ClientID%> :checkbox + a").attr("href", "#");
    $("#<%=tvProducts.ClientID%> :checkbox").click(function() {
    if ($(this).is(":checked")) {
    //勾选时勾选所有父节点
    $(this).parents("div").prev().find(":checkbox").attr('checked', true);
    }
    else {
    //未勾选时所有子节点未勾选
    $(this).closest("table").next().find(':checkbox').attr('checked', false);
    }
    });
});
</script>
```
---
从我的百度空间导入