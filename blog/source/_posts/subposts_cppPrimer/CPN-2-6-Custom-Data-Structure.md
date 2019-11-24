---
title: 'CPN 2-6: Custom Data Structure'
date: 2019-11-24 12:04:25
categories: cppPrimer
tags: cpp
---

# Defining Our Own Data Structure

通常来说，一个data structure指的是如何组织相关数据的方法和如何操作这些数据的策略。在C++中，我们通过`class`或者`struct`来定义我们自己的数据结构：

```c
struct Sales_data{
  string bookNo;
  unsigned units_sold = 0;
  double revenue = 0.0;
};  // must have semicolon
```

在这里我们通过`struct`定义了一个`Sales_data`的数据结构，其中大括号里面定义的变量称为`data members`（数据成员）。一个类中的数据成员定义了这个类的对象中的内容。类的每一个对象都有自己的数据成员的copy。**一个类中的数据成员中的每个变量都不能重复**，但是可以和这个类的作用域以外的变量同名（不属于这个scope）。

需要注意的是，这里我们只是在类中定义了数据成员，而没有定义operation和函数，实际上一个类中还可以重载操作符和定义函数。因为这一章并不是主要讲解类（class）的使用，所以此时我们只使用了`struct`来做演示。这里的`struct`实际上更像是C的用法，因为其只定义了`data member`。但是在C++中，`struct`实际上也是定义了一个类，也可以继承，也可以定义构造函数等，所以C++中的`struct`是C中的`struct`的扩展与改进。关于`struct`和`class`的区别，也可以查看[here](https://www.hellscript.cc/2019/02/23/subposts_c/struct-VS-class/)。

> **Tips:**
>
> - 类的定义通常写在头文件中，并且这个头文件通常以这个类的名字命名。
> - 一旦头文件变化，包含这个头文件的源文件都会被重新编译。这也是为什么函数一般不会定义在源文件，因为函数经常变化，这样会导致编译效率降低。
> - 头文件通常包含`entities`（因为只能被定义一次），例如`definition`，`const`和`constexpr`变量。因此需要preprocessor来定义`header guards`来保证头文件只被展开一次。**头文件都应该增加`header guards`**，哪怕这个头文件不被其他源文件包含。

`Header guards`的实现依靠`preprocessor variable`。一个`preprocessor variable`有两个状态：`defined`和`undefined`。一个`header guards`的例子如下：

```c
#ifndef __TEST_H__  
// true if the variable `__TEST_H__` has not been defined
#define __TEST_H__
// define the preprocessor variable `__TEST_H__`

extern const int ci = 42;

#endif
```

`#ifndef`用于判断这个变量`__TEST_H__`是否已经被定义，如果没定义则为`true`，继续往下执行，即接着`#define __TEST_H__`定义这个变量，且执行我们真正想要在头文件中定义的变量的代码；否则会跳到`#endif`。需要注意的是`preprocessor variable`并不是我们自己C++程序本身的变量，所以并不遵循C++ scoping rules。并且`preprocessor variable`在整个程序中都必须unique，所以在实践中我们通常会把它定义为`__[header name]_H__`。

因为之前需要些很多coursework，所以我写了一个mac下的[C项目模板快速生成脚本](https://github.com/gzrjzcx/project_templete)，包含了用于编译的`Makefile`，以及快速创建`.h`文件和`.c`文件的脚本，还有在submit作业的时候进行检查的脚本，看是否有相应的文件遗失。其中快速创建的`.h`文件都是包含了`header guards`。