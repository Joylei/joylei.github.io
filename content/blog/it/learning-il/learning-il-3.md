+++
title = "IL学习(三)"
date=2010-05-05

[taxonomies]
categories=["Programming"]
tags=[".Net", "IL"]
+++

## 原始字符串
IL中没有原始字符串，如C#中的@"\"在IL中则是已经经过编译器转义的字符串"\\"。

## 类的继承
IL使用extends关键字表示类的继承
```
.assembly my{}
.class auto ansi Animal
{
.method static void main() il managed
{
    .entrypoint
    newobj instance void Dog::.ctor()
    callvirt instance void Animal::run()
    ret
}
.method instance void .ctor() il managed
{
    ret
}
.method virtual instance void run() il managed
{
    ldstr "an Animal is running"
    call void [mscorlib]System.Console::WriteLine(string)
    ret
}
}
.class auto ansi Dog extends Animal
{
.method instance void .ctor() il managed
{
    ldarg.0
    call instance void Animal::.ctor()
    ret
}
.method virtual instance void run() il managed
{
    ldstr "an Dog is running"
    call void [mscorlib]System.Console::WriteLine(string)
    ret
}
}
```

子类的构造函数必须调用父类的构造函数，否则会报InvalidProgramException异常。
实例方法如果没有呢new/virtual关键字，则默认为new，virtaul则标明为虚方法，表明方法可以重写或覆盖了父类的方法（在C#中用override）。
callvirt：在运行时决定要调用的方法。

## 实现接口
```
.assembly my{}
.class interface IRunnable
{
.method public virtual abstract void run() il managed
   {}
}
.class auto ansi Animal implements IRunnable
{
.method static void main() il managed
{
    .entrypoint
    newobj instance void Dog::.ctor()
    callvirt instance void Animal::run()
    ret
}
.method instance void .ctor() il managed
{
    ret
}
.method public virtual newslot instance void run() il managed
{
    ldstr "an Animal is running"
    call void [mscorlib]System.Console::WriteLine(string)
    ret
}
}
.class auto ansi Dog extends Animal
{
.method instance void .ctor() il managed
{
    ldarg.0
    call instance void Animal::.ctor()
    ret
}
.method public virtual instance void run() il managed
{
    ldstr "an Dog is running"
    call void [mscorlib]System.Console::WriteLine(string)
    ret
}
}
```

实现接口时如果方法签名不匹配或为实现接口中的方法，将报TypeLoadException异常。
newslot:当第一次把方法声明为virtual方法时，最好把它标记为newslot，它表明虚方法的根。

## 析构函数
C#中的析构函数编译成IL后，转化成Finalize()方法。
```
.assembly my{}
.class interface IRunnable
{
.method public virtual abstract void run() il managed
   {}
}
.class auto ansi Animal implements IRunnable
{
.method static void main() il managed
{
    .entrypoint
    newobj instance void Animal::.ctor()
    callvirt instance void Animal::run()
    ret
}
.method instance void .ctor() il managed
{
    ret
}
.method public virtual newslot instance void run() il managed
{
    ldstr "an Animal is running"
    call void [mscorlib]System.Console::WriteLine(string)
    ret
}
.method family hidebysig virtual instance void Finalize() il managed
{
    ldstr "in class Animal destructor"
    call void [mscorlib]System.Console::WriteLine(string)
    ret
}
}
```
虽然Finalize的执行是由运行时决定的，但是在这个例子中还是可以看见执行的。
类不会继承构造函数和析构函数。

## 访问修饰符internal
C#的访问修饰符internal只是C#词法的一部分，与IL没有任何关系。
C#中父类的可访问性不能比子类小，但是在IL中不遵循这个规则。
C#中方法的可访问性受它所在的类的可访问性的限制，但是在IL中没有这个限制。

## 强制类型转换
```
.assembly my{}
.class interface IRunnable
{
.method public virtual abstract void run() il managed
   {}
}
.class auto ansi Animal implements IRunnable
{
.method static void main() il managed
{
    .entrypoint
    .locals(class Animal V_0,class Dog V_1)
    newobj instance void Dog::.ctor()
    stloc.0    //转换成Animal
    ldloc.0
    callvirt instance void Animal::run()
    ldloc.0
    castclass Dog //转换成Dog
    call instance void Dog::run()
    ret
}
.method instance void .ctor() il managed
{
    ret
}
.method public virtual newslot instance void run() il managed
{
    ldstr "an Animal is running"
    call void [mscorlib]System.Console::WriteLine(string)
    ret
}
.method family hidebysig virtual instance void Finalize() il managed
{
    ldstr "in class Animal destructor"
    call void [mscorlib]System.Console::WriteLine(string)
    ret
}
}
.class auto ansi Dog extends Animal
{
.method instance void .ctor() il managed
{
    ldarg.0
    call instance void Animal::.ctor()
    ret
}
.method public virtual instance void run() il managed
{
    ldstr "an Dog is running"
    call void [mscorlib]System.Console::WriteLine(string)
    ret
}
}
```
如果转换失败，将抛出InvalidCastException异常。

C#中不能继承标明Sealed的类，但在IL中可以。
```
.assembly my{}
.class private auto ansi sealed zzz extends System.Object
{
    .method public hidebysig static void vijay() il managed
    {
        .entrypoint
        ret
    }
}
.class private auto ansi yyy extends zzz
{
}
```

## literal

C#编译器会在编译时计算所有常量的值。literal字段代表常量，但是不可以在IL中访问，如果访问会在运行时报错MissingFieldException。

## initonly
C#中用readonly标记只读字段，在IL中使用initonly。

---
从我的百度空间导入