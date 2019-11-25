---
title: 'C++Primer 3-3: Vector'
date: 2019-11-25 16:00:53
categories: cppPrimer
tags: cpp
---

# Library `vector` Type

> a `vector` is a collection of **objects**, all of which have the **`same type`**.

在C++中，一个`vector`就是`一系列相同类型的objects`。通常，`vector`也被称为容器(container)，因为它就像是一个容器来盛放其它objects。

`vector`是一个类模板（class template），因此我们在使用的时候必须要提供额外的信息来创建这个类。`模板（templates）`不是方法（function）或者类（class），而可以看作是指导compiler如何产生类或函数的**指令**（instructions），或者说规则。而compiler从模板中产生类或者方法的过程叫作`实例化（instantiation）`。当我们要使用一个目标来实例化类的时候，需要指明（即提供额外信息）我们想要实例化哪一种类或方法。模板是一个比较复杂的东西，我们会在之后详细说到它。

> **Tips:**
>
> - `vector`是一个`template`，不是一个type。但是从`vector`模板中生成的包含element type的如`vector<int>`则是type。
> - 因为`reference`不是object，所以`vector`并不能用来盛放`reference`。

## `vector` Definition and Initializition

与其他class type一样，`vector`有很多种初始化方法，同时C++11还提供了`list initialize`：

```c
vector<int> iv;  // definition, iv is an empty vector
vector<int> iv2(iv);  // copy elements of iv into iv2
vector<int> iv2 = iv;  // equivalent to the above
vector<string> sv(iv);  // error: sv holds strings not int;

vector<int> iv3(10, -1);  // ten int elements, each initialized to -1
vector<int> iv4(10);  // ten elements, each initialized to 0
vector<string> sv2(10, "hi!"); // ten string elements, each initialized to "hi!"
vector<string> sv3(10); // ten string elements, each initialized to empty

/*  list initializer  */
vector<string> v1{"a", "an", "the"};
// three elements, the first holds "a", the second holds "an", the third holds "the". Must use curly braces.

```

