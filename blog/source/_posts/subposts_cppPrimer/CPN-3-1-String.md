---
title: 'CPN 3-1: String'
date: 2019-11-24 13:19:35
categories: cppPrimer
tags: cpp
---

# String

在C++中，除了内置的基本类型，还有很多库定义了抽象数据类型。其中最重要之一就是`string`，支持可变长度字符串（variable-length character strings）。

## 1. Namespace `using` Declarations

> `scope operator(::)` tells compiler to look in the scoop of the left-hand operand for the name of the right-hand operand.

范围操作符（`::`）会告诉compiler去哪个namespace寻找变量的定义。

我们可以在文件头使用`using`关键字来声明我们想要用的namespace members，这样在后续的代码中都不需要在指定`cin`或者`cout`是来自`std namespace`：

```c
using std::cin;
using std::cout;
```

也可以直接在程序中通过`::`来指定namespace：

```c
std::cout<<"Hello world"<<std::endl;
```

注意通常我们**不能**在头文件中使用`using`关键字来指明想要使用的`namespace`，因为头文件会被展开到包含它的源文件，这样每个包含它的源文件就会使用到这个`namespace`里面所定义的operation，这是不对的。

> **Tips:**
>
> 不要在头文件（`.h`）文件中使用`using`来指定`namespace`。

## 2. Library `string` Type

> A `string` is a **variable-length**（可变长度）sequence of characters.

要使用`string`type，我们必须包含`string`头文件。且因为`string`是库的一部分，被定义在`std`namespace中，所以在我们代码中如果想要使用`string`，需要如下代码：

```c
#include <string>
using std::string;
```

### `string` Definition and Initializing

`string`的初始化可以分为两种：

- `copy initialize`：通过赋值操作符`=`将`right-hand initializer` **copy** 到被创建的`left-hand variable`。新创建的object是initializer的一份copy。复制初始化总是调用复制构造函数。
- `direct initialize`：不通过赋值操作符`=`，而是通过调用与实参相匹配的构造函数。

关于`copy initialize`和`direct initialize`的具体区别，可以查看[here](https://blog.csdn.net/ljianhui/article/details/9245661)。因为其中涉及到构造函数等知识，会在后面详细提到。`string`具体的定义与初始化例子如下：

```c
string s1; // default initialization; s1 is an empty string
string s2 = s1; // copy initialize
string s2(s1); // direct initialize
string s3 = "hello world"; // copy initialize; s3 is a copy of the literal
string s3("hello world"); // direct initialize; s3 is a copy of the literal
string s4(10, 'c'); // direct initialize; s4 has 10 copies of the character 'c'
```

注意`s4`的这种初始化方法只能通过`direct initialize`实现。即最终得到`s4`为`cccccccccc`。

### `string` Operations

常见的`string`操作总结如下表：

| Operations                | Meaning                                                      |
| :------------------------ | :----------------------------------------------------------- |
| os << `s`                 | Writes `s` onto output string os. Return os.                 |
| is >> `s`                 | Reads `whitespace-separated` string from `is` to `s`. Return is. |
| getline(is, `s`)          | Reads a `line` of input from `is` to `s`. Return `is`.       |
| `s`.empty()               | Returns `true` if `s` is empty; otherwise return `false`.    |
| `s`.size()                | Returns the number of characters in `s`. Compatible with STL container. |
| `s`.length()              | Equivalent to s.size(). Compatible with C style.             |
| `s`[n]                    | Returns a `reference` to the `char` at position of `n` in s. `**Position starts at 0.**` |
| `s1` + `s2`               | Returns a string that is the `concatenation` of s1 and s2.   |
| `s1` == `s2`              | s1 and s2 are equal if they `contain the same characters`. **Case-sensitive**. |
| `<`,`<=`,`>=`,`>`         | Comparisions are `case-sensitive` and use `dictionary ordering`. |
| `s`.substr(start, length) | Returns a substring in `s`, from `start` to `start + length`. |

上述常见操作需要注意的几点：

- `cin>>s`读取`stdin`输入是`whitespace-separated`，即碰到空白字符就会结束，如果想要读取一整个句子（即忽略空白符），要使用`getline()`。
- `s.size()`和`s.length()`都是得到`s`的字符个数，只不过`size()`是为了兼容STL容器习惯，`length()`是为了兼容C的`strlen()`习惯。
- 两个字符串相加相当于合并两个字符串，**`string`并没有重载`-`操作**。想要得到一个`string`的子串可以使用`substr()`。

### `string` IO

在OJ模式的编程题中（例如牛客网），我们需要自己处理输入输出，所以在此说一说`string`如何处理输入输出：

```c
string s;
while(cin>>s)     // read until end-of-file, whitespace is separator
  cout<<s<<endl;

while(getline(cin, s))    
  // read input a line at a time until end-of-file, ignore whitespace
  cout<<s<<endl;
```

注意这里需要说明一下为什么可以通过`cin`来判断是否到达end-of-file。首先，`cin`是一个`istream`类型的object，其用于从输入流中（`stdin`）读取数据。然后，在`istream`类中对操作符`>>`进行了重载，分别定义了包括`bool`，`int`，`unsigned`等在内的多种格式化输入。详细地说，操作符`>>`被定义为接受两个operands（操作对象）：

- left-hand operand必须是一个`istream`类型的object，即`cin`。
- right-hand operand是一个接收`cin`中的值的object，即`s`。

所以，`>>`操作符的含义就是从左边的`cin`中读取标准输入流中的值，并且赋值给右边的`s`，同时**`return left-hand operand，即返回istream类型的对象cin`**。因为input operator(i.e. `>>`)返回值为`cin`（注意是`>>`操作符的返回值而不是`cin`的返回值，`cin`本身是一个object，不存在返回值），所以可以在一条单独的语句中进行多次读取操作：

```c
std::cin >> v1 >> v2;
// equavilent to below:
(std::cin >> v1) >> v2;
```

简单来说，就是`std::cin`中有两个输入值，第一个`>>`操作符从`cin`中读取第一个输入值并赋给`v1`，第二个`>>`操作符从`cin`中读取第二个输入值并赋给`v2`。

同时，流`istream`类中定义了4个标志位来表示流的状态：`badbit`，`eofbit`，`failbit`和`goodbit`。当从流中读取到`EOF`结束符时则会将`eofbit`置为`ture`，或者读取到错误数据（例如类型不一致）等，`failbit`也会被置为`true`，这个时候`cin`作为返回值其结果不再是`true`，所以整个输入流程会结束。具体可以看[here](https://blog.csdn.net/hou09tian/article/details/78335548)。

### `string::size_type` Type

 为了能够在各种不同类型的机器上使用library types，一个类通常会定义一些伴生类型（companion types）。例如，`string::size_type`就是一个伴生类型，尽管我们不能知道这个类型的精确定义，但是我们知道它是一个`unsigned type`，以保证它足够大来保存任何`string`的size。我们可以直接定义这个类型，也可以让compiler帮助我们推断：

```c
string::size_type len = s.size();
auto len = s.size();
```

需要注意的是因为`len`是`unsigned`，所以如果我们在进行`if(len < n)`比较的时候，如果`n`是`int`类型且为负数，那么会发生隐式类型转换，`n`会变为最大的`unsigned`，也就是说这个条件判断永远为`true`。

### `string` Comparison

从上面的`string`常用操作表中可以看到`string` class重载了比较操作符，这也就说明比较两个`string`是有意义的：

- 两个`string`相等指的是这两个`string`的**`size相同，每个字符也相同（case-sensitive）`**。
- 字符串`s1 > s2`的意义为：
  - `s1`与`s2`的第一个不相同的字符`c1 > c2`(ascii code)。
  - `s1.size() > s2.size()`且较短的字符串`s2`与较长的字符串`s1`的子串相等。

> **Conclusions:**
>
> 两个字符串比较即比较其每一个字符。谁的第一个不相同的字符大谁就大。

### `string` Addition

两个`string`结合可以直接使用重载的操作符`+`，非常方便。但是将一个`literal`与`string`相加则有可能出现问题：

```c
string s1 = "hello", s2 = "world";
string s3 = s1 + ',' + s2 + '\n';  // ok
string s4 = s1 + ", ";  // ok, adding string s1 and a literal
string s5 = "hello" + ", ";  // error: two literal cannot add
string s6 = s1 + ", " + "world";  // ok, s1 is string
string s7 = "hello" + ", " + s2; // error: "hello" cannot add to ", "
```

其中`s7`不能编译通过是因为这个表达式的运算顺序为`("hello + ", ") + s2`，第一个加法操作是没有意义（不能操作，编译器不知道如何操作）的。因为对于`string literal`，是没有重载`+`操作符的。

>**Conclusions:**
>
>只有`string`才能进行加法操作，这要求`+`操作符两边至少有一个是`string`类型。