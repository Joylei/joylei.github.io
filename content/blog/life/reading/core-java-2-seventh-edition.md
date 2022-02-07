+++
title = "Core Java 2 Seventh Edition 读书笔记"
date=2007-08-17

[taxonomies]
categories=["Programming", "Reading"]
tags=["reading", "java"]
+++

Java中的字符一般是16位的，即utf16，例如“我爱你hello”字符个数为8。但是有些额外字符是需要32位，高位在前16位，低位在后16位。为了避免出错，尽量不要对字符操作。

要改变java中的一个字符串，最好是创建一个新的字符串，(原因是编译器可以将字符串共享)。

检测字符串相等用eques或compareTo方法。

标准输入流System.in,

Scanner in = new Scanner(System.in);

in.next();读入一个单词

in.nextLine();读入一行

in.nextInt();读入一个整数

格式化输出用System.out.printf();用法同C类似。String.format()类似sprintf()。

不能在嵌套的块中声明同名的变量。

Java中没有elseif,而是嵌套的if…else。

Case标签必须是整数或者枚举常量，不能是字符串。

Break很强大，可以使用数字，表示跳出第几层循环，还可以使用标签，表示跳出块或循环。

数组拷贝(=)是引用，要复制所有元素，应使用System.arrayCopy(from,fromIndex,to,toIndex,count)。

未完待续...

---
从我的百度空间导入