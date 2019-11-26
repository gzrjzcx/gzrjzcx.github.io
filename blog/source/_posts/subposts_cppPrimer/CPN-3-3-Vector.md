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

与其他class type一样，`vector`有很多种初始化方法，同时C++11还提供了`list initializer`：

```c
vector<int> iv;  // definition, iv is an empty vector
vector<int> iv2(iv);  // copy elements of iv into iv2
vector<int> iv2 = iv;  // equivalent to the above
vector<string> sv(iv);  // error: sv holds strings not int;

/*  parentheses initializer (count, value)  */
vector<int> iv3(10, -1);  // ten int elements, each initialized to -1
vector<int> iv4(10);  // ten elements, each initialized to 0
vector<string> sv2(10, "hi!"); // ten string elements, each initialized to "hi!"
vector<string> sv3(10); // ten string elements, each initialized to empty

/*  list initializer  */
vector<string> sv4{"a", "an", "the"};
// three elements, the first holds "a", the second holds "an", the third holds "the". Must use curly braces.
vector<string> sv4("a", "an", "the");  // error: must use {}

/*  mixed curly braces initializer {} and parentheses initializer ()  */
vector<int> iv5(10);  // ten elements with value 0
vector<int> iv6{10};  // one element with value 10
vector<int> iv7(10, 1);  // ten elements with value 1
vector<int> iv8{10, 1};  // two elements with value 10 and 1
vector<string> sv5{"hi"};  // list initialization, one element
vector<string> sv5("hi");  // error: can't construct a vector from a string literal
vector<string> sv6{10};  // ten elements with empty string
vector<string> sv7{10, "hi"}  // ten elements with value "hi"
```

首先，如果使用一个`vector`来初始化另一个`vector`，不管是`copy initialization`还是`direct initialization`，其含义都是**拷贝initializer中所有的elements到新创建的这个vector中**。注意新vector中的每一个元素都是initializer中对应的每一个元素的`copy`，而不是`reference`。

因为C++11中新增了`list initializer`，即使用curly braces`{...}`来作为initializer，所以要区分其与`direct initializer`（即使用parenthesis`()`）的区别。尤其是当initializer和vector的类型不一致的时候。例如，`sv5`，`sv6`和`sv7`都使用了`{}`，但是只有`sv5`是`list initialization`。**因为并不能使用`10`作为initializer value来初始化`sv6`，所以此时compiler会尝试使用`parenthesis initializer`的方法来初始话这个vector。**

> **Conclusions:**
>
> - `list initializer`表示的是使用`{}`中的每一个value来进行初始化，**`{}`中有几个值则vector中有几个elements，每个element的值是`{}`中对应的value。**
> - `parenthesis`表示的是使用`in-class initializer`即`()`作为initializer。`()`中有两个参数：`(count, value)`。
> - 当使用`list initializer`来初始化，但是`{}`中的值并不能初始化vector时，即`vector<type>`中的`type`与`{}`中的值的type不一致时（例如`sv6`），compiler则会考虑使用其他初始化方法。这种情况只针对`list initializer`，对于`parenthesis`则不会，类型不对时则会直接报错。