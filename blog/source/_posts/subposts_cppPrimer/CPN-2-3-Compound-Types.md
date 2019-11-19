---
title: 'C++Primer 2.3: Compound Types'
date: 2019-11-12 16:49:07
tags: cpp
categories: cppPrimer
---

# Chapter 2-3: Compound Types

> Compound types is a type that is defined in terms of another type.

复合类型指的是根据另一种类型定义的数据类型，C++中最常见的两种就是引用（reference）和指针（pointer）。注意一般情况下我们说的reference，都是`lvalue reference`。

## 1. Reference

> A `Reference` defines an alternative name for an object.  A reference is an `Alias`.

`引用`定义了一个object的`别名`。可以将引用看作是这个object。**`一个reference不是一个object，一个reference只是另一个已经存在的object的别名，因此引用不需要内存（这里存疑），也没有内存地址`**。当我们定义一个引用的时候，并不是将intializer的值copy给变量，而是`将引用绑定（bind）到initializer`。引用的定义示例如下，注意一个常见的declaration具有如下格式：`base type + declarator + identifier = initializer;`

```c
int i = 1024, j = 2048;
int &ri = i;  //ri refers to i, a declaration includes:
// base type(int) declarator(&) identifier(ri) = initializer(i)
&ri = j // error: a reference cannot be rebind a different object
int &ri2; // error: a reference must be initialized
int &ri3 = NULL; // error: a reference cannot be NULL
double &ri4 = i; // error: a reference type must mactch exactly
int &ri5 = 10; // error: a reference must be initialized to an object
int &ri6 = ri // ok, ri4 refers to the i(ri is bound);
int &ri7 = i, &ri6 = j; // ok, two references need two declarator respectively
```

注意`引用不能为空`这个说法比较模糊，准确地说应该是`引用`不能bind to空指针指向的值（会造成undefined行为），但是`引用`可以bind to空指针，详情[here](https://bbs.csdn.net/topics/370082407)。另外，在定义`引用`的过程中不会用到address-of操作，即`&`不会出现在右值上。

## 2. [Pointer](https://www.hellscript.cc/2019/02/22/subposts_c/The-nature-of-pointer/)

> A pointer holds the address of another object.

可直接点击标题查看pointer的详细说明。这里有一个编程建议就是对于指针：`Initialize all pointers`。

## 3. Compound Type Declarations

在一个变量的声明中往往会有一个base type加上一列declarators，每一个变量都可以使用自己的declarator来决定这个变量是什么类型（指针或引用）。这里有一个coding建议就是：

```c
int* p; // ok, a pointer p
int* p1, p2; // ok, p1 is a pointer and p2 is int
int *p1, p2; // ok, p1 is a pointer and p2 is int
```

如上述例子，`int* p1, p2`是很容易造成误解的一种写法，这会让人觉得所有的identifier（p1 和 p2）都是被declarator修饰，然而实际上只有第一个identifier（p1）被declarator修饰，所以第二个identifier（p2）是int类型而不是指针。所以， 推荐使用下面那种定义方式，即将declarator和identifier放在一块。`每个identifier都有自己的declarator`。

另外，我们有指针的指针（a pointer to a pointer），有指针引用（reference to pointer），但是`没有引用指针（pointer to reference）`。因为`a reference is not an object`，在内存中没有地址，所以指针不能指向一个引用。下面是一个指针引用的例子：

```c
int *p;
int *&r = p; // r is a reference to a pointer(the pointer = p)
```

注意这里是`从右往左`读，**而不是**像我们习惯性地读作：~~`&*r` a reference（&）to a pointer（*r）~~。

## 4. Pointer vs. Reference

1. 从语言层面上看，`引用`可以看作是`const指针`，即引用必须初始化，且不能初始化为0（NULL），且引用初始化bind过后就不能再rebind到另一个不同的object上。对引用的操作都可以看做是对绑定对象的操作。

2. 从内存分配上看，`指针`是一个真实的object，系统需要为其分配内存来存储它。但是`引用`是一个object的别名，从理论上说`引用` 并不需要分配内存。**但是，在当前很多编译器对于`引用`的implementation，都是`通过指针来实现引用`**。所以`引用`到底需不需要分配内存，取决于编译器（或者说取决于`引用` 如何实现），因为C++标准并没有规定引用如何实现。在我自己64位的MAC上测试，使用icc编译器：

   ```c
   struct test{
     int &r;
   }
   cout<<sizeof(int &)<<endl; // 4
   cout<<sizeof(test)<<endl; // 8
   ```

   在64位机器上`sizeof(int) == 4`，所以`sizeof(int &) == 4`，因为`sizeof（引用类型）`得到的结果是引用类型的大小。但是`sizeof(test)`得到的结果却是8，这说明`test`里面是一个`指针`的偏移量。也就是说，在我的icc编译器上`引用`是通过`指针`来实现的，所以也要给它分配内存空间，[here](https://bbs.csdn.net/topics/320095541)。

3. 从编译上看，在编译时会分别将`指针`和`引用`都添加到符号表上，符号表上记录的是变量名和对应的地址。`指针`变量在符号表上的地址值为`指针变量的地址值`，但是`引用`变量在符号表上的地址值为`所引用对象的地址值`。

4. 实际使用上，`引用`因为其`const`的属性，所以不会为NULL，所以作为函数形参的时候也不需要进行空值检查，所以相对会更方便安全。