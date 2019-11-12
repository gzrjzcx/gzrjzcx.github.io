---
title: 'C++Primer 2.2: Variables'
date: 2019-11-12 14:21:29
tags: cpp
categories: cppPrimer
---

# Chapter 2.2: Variables

> A variable provides us with named storage that our programs can maniplage.

变量给我们提供了一个程序可以操作的命名存储内存。每个变量都有类型（即上篇笔记提到的type）。变量的`类型（type）`决定了这个变量的`size`（即在内存中占多少字节）,`layout`（即在内存中的存储布局），该变量能够存储的`value` 以及能够应用在此变量上的`operations`。

注意，在此书中`object`与`variable`是可以互换的概念。`object`指的是一片包含数据且具有类型的内存区域，而不单单是指`class`实例化出来的对象。

> An `object` is a region of memory that can contain data and has a type.

## 1. Variables Definitions

变量的定义（definition）总是包含着初始化（intialization）。如果给定了初始化的值（initializers），则被定义的变量将会在**它被创建的时候**得到这个值；如果没有给定初始化的值（without initializers），则被定义的变量将会被`default initialized`，也就是说被定义的变量将会被给予默认值。C++内置的全局变量（定义在函数外的变量）将会被初始化为0，C++的内置的局部变量（定义在函数内的变量）则是`uninitilized`，它的值是`undefined`。变量定义的例子：

```c
double price = 109.99, discount = price * 0.16; // ok
```

注意在同一个定义中，用一个更早定义的变量来初始化后面定义的变量是可行的。即在上述例子中`discount`可以用来初始化`price`，因为`discount`此时已经被定义完成且其值为initializer。正如之前我们所说，初始化会让变量在其被创建的时候得到initializer的值（初始值）。变量的定义包括如下3个动作：

- 指明变量的名字（name）和类型（type）。
- 给变量申请内存空间。
- `(Optional)`指定初始值（initializer），没有的话则是默认初始值。

另外，需要注意的是变量定义中的初始化（initialization）和赋值（assignment）是不同的。

> 初始化指的是在变量`被创建时`给定其初始值（initializer），赋值则指的是消除一个变量当前的值，并用新的值来取代该值。

## 2. Variable Declarations

变量的声明（declaration）与变量的定义（definition）有着本质的区别。变量的声明主要是为了告诉编译器这个变量已经被定义过了，使用别的地方的定义来解释此变量。因为C++支持单独编译（separate compilation），所以变量的声明主要是为了`share code across those files`。变量的声明其实就是变量定义的第一个动作，即指明变量的名字（name）和类型（type），并且告诉编译器此变量已经被定义在其他文件。变量定义与声明的区别如下：

> - 定义包含了声明。声明只是指明了变量的名字和类型，但是定义还为变量申请了内存空间，并且可能赋予初始值（即创建了相关的实体（entity））。
> - 一个变量只能有一个定义，可以有多个声明（实际上每个include定义某个变量头文件的文件都需要声明此变量）。

另外，需要注意的是，变量的声明（declaration）只有一种方式：

```c
extern int i; // declares but does not define i
int j; // declares and defines j
extern double pi = 3.1416; // definition
```

如上述例子，只有使用`extern`关键字修饰**且没有提供`initializer`**的方式算作是变量的声明，其余均是变量的定义。即使是使用了了`extern`关键字，但是同时提供了`initializer`，`extern`关键字也会被重载从而变成变量定义。总结就是：**只要有initializer，都是定义；只有extern且没initializer才是声明**。

## 3. Variable Scope

在编程的语境（context）中，`a scope is a part of the program in which a name has a particular meaning`。因此，我们需要注意所定义的变量是在什么范围内有效。在C++中，通常使用一对大括号定义一个scope，在这个scope内声明的变量都只会在这个scope内有效（注意此时的有效指的是visable, 即编译器能够找到它的定义）。例如：

```c
int reused = 42;
int main()
{
  int reused = 0;
  cout<<reused<<endl;  // 0
  cout<<::reused<<endl; // 42, globla scope has no name
}
```

虽然定义了全局变量`reused`，但是也定义了一个局部变量`reused`在`main`这个scope内，所以在`main`这个scope内`reused`指的是local的变量。但是需要注意的是，`global::reused`这个变量并不是被`local::reused`替换了，只是在`main`这个scope内`global::reused`这个变量不可见了。但是我们依然可以强行调用它，只需要知道它是哪个scope即可。注意全局的scope没有名字，所以调用全局scope的变量可以直接用scope操作符，即`::reused`。