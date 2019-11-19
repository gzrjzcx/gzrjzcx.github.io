---
title: 'C++Primer 2.4: const'
date: 2019-11-19 16:46:47
categories: cppPrimer
tags: cpp
---

#  `const` Qualifier

`const`是个比较复杂的关键字，它的需求来自于：当我们需要定义一个`其值不会改变的变量（Variable）`时，就需要用到`const`关键字：

```c
const int bufSize = 512; // ok, initialized at compile time
const int bufSize = get_size(); // ok, initialized at run time
const int bufSize; // error, const must be initialized
```

`const`定义的变量必须进行**初始化**，因为`const`变量在创建之后就不能更改了。其实现是在编译阶段时compiler会将`const`类型的变量替换为其对于的值（即initializer）。例如，对于上述代码，编译器会用`512`替换`bufSize`。

## 1. `const`变量的scope

> By default, `const objects` are **local to a File**. 

### 为什么const变量是文件级局部变量？

因为compiler在编译阶段需要将const变量替换为其初始化的值（initializer），所以compiler必须能够知道这个initializer。为了让compiler知道这个initializer，这个const变量在每个使用到它的File中都需要定义。因此，为了让compiler知道const变量的initializer，同时避免重复定义（duplicate definition）的错误，`const变量被定义为local to the file`。也就是说，如果我们在不同的Files中定义一个同名的const变量，这个变量在每个File中都是一个单独的变量。

如果想定义一个`全局的const变量`（a single instance of a const Variable），我们可以使用`extern`关键字。**注意，对于全局`const`变量，其定义（definition）和声明（declaration）都需要明确使用`extern`。**

```c
// main.c defines and initializes a const variable that is accessible to other files
extern const int bufSize = f();
// file_1.h declares the const variable
extern const int bufSize;
```

