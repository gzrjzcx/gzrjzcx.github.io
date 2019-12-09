---
title: 'C++Primer 3.5.1: C-style character string'
date: 2019-12-09 13:45:32
categories: cppPrimer
tags: cpp
---

# C-Style Character Strings

> `C-Style strings` are not a type. They are a **convention** for how to represent and use character strings.

`C-Style strings`指的是一种指定如何表示和使用`character strings`的规定。该规定要求：

- `character strings`存储在`character array`中；
- `null terminated`。即该`character array`最后一个element为`null character '\0'（ansii = 0）`。

例如，C++中的`string literals`就是一个`C-Style string`（从C中继承而来），也就是说，在C++中`string literals`是`null terminated`。注意在这里出现了几个很容易混淆的定义，我们需要通过实际例子来辨析：

```c
'a'  // a character 'a'
"hello, world";  // a string literal, has '\0'
char sa[] = {'h','e','l','l','o'};  // sa is a character array, no '\0'
char *p = "hello, world";  // p is a pointer to a string literal "hello, world"， p points to the first element of this string literal
string s("hello, world");  // s is a string(library type)
```

首先，`string literals`指的就是一个字符串字面量，其是一个常量，存储在`DS`段。因为C/C++中的`string literals`都遵循`C-Style Strings`规定，因此C/C++中的`string literals`都是存储在一个`character array`中，且这个`character array`的最后一个元素是`'\0'`。

其次，`sa`是一个`charater array`数组，也就是常说的字符数组（`char数组`）。数组中的每个element都是一个`character`，也就是常说的字符（`char`）。字符数组并不遵循`C-Style strings`规则，所以其并不要求最后一个元素是`'\0'`，即并不是`null terminated`。但是需要注意的是，尽管`sa`在进行操作时会被compiler转换为一个`指向首元素的指针`，但是**其是一个char * const的指针，即指针自身不能被修改**，所以并不能给`sa`赋值。同理，也不能再次使用`string literals`给`sa`赋值（注意这里如果使用`string literals`给`sa`进行初始化是可以的）。

类似的，`p`是一个`char`类型的指针，也就是常说的`char *`，其指向一个`string literals`常量，更准确的说是指向该`hello, world`的首元素（即`h`字符）。因为`string literals`是一个`character array`，所以可以通过这个指向首元素的指针来实现`subscript`操作。但是需要注意的是，因为此时`hello, world`是一个常量，所以尽管可以通过取下标获得其中某个element，但是并不能修改这个element。更多关于`char*`和`char[]`的区别可以查看[here](https://www.hellscript.cc/2019/02/22/subposts_c/charPointer-VS-charArray/)。

最后，就是`string`类，其是`library type`，相应的操作符都在类中被重载。注意，在C++11标准中，明确规定了`string`也是`null terminated`，即以`'\0'`结尾，[here](https://www.zhihu.com/question/33312840)。

## C library string functions

因为C standard library并没有重载诸如`+`，`==`等操作符，所以并不能用C++中的`string`类中重载这些操作符对`C-Style string`进行操作。相反，C中提供了一系列functions来操作`C-Style string`。例如：

| Functions        | Meaning                                                      |
| ---------------- | ------------------------------------------------------------ |
| str`len`(p)      | Returns the length of `p`, **not couting the null**          |
| str`cmp`(p1, p2) | Compares `p1` and `p2` for equality. Returns 0 if `p1 == p2`, a positive value if `p1 > p2`, a negative value if `p1 < p2`. |
| str`cat`(p1, p2) | Appends `p2` to `p1`. **Returns `p1`**.                      |
| str`cpy`(p1, p2) | Copies `p2` into `p1`. **Returns `p1`.**                     |

**其中这些函数的实参必须是指向`null terminated array`**的指针。如果传入的指针不是指向`null terminated array`，那么会出现`undefined`的情况。例如`strlen`，则会不断循环直到找到`null`。

在C++中，因为`string`已经重载了很多操作符，所以使用起来很方便。例如，当我们想拼接两个字符串时：

```c
/*  c++  */
string s1("hello");
string s2("world");
string largeStr = s1 + ' ' + s2;

/*  c  */
char s1[] = "hello";
char s2[] = "world";
char largeStr[12];
strcpy(largeStr, s1);
strcat(largeStr, " "); // can't use ' ', because " " is a string literal, can be converted to pointer 
strcat(largeStr, s2);
```

可以看到，在C中想要实现拼接字符串非常麻烦，需要定义一个`largeStr`来接收拼接好的字符串；并且还需要计算好新的字符串的长度。但是在C++中如果使用`string`类的话则可以直接使用重载的`+`进行操作。

## Mixing `library strings` and `C-Style Strings`

通常来说，在任何要求使用`string literal`的地方，我们都可以使用`null-terminated array`。但是反过来却不行，即我们不能在要求使用`C-Style Strings`的地方使用`library strings`。

```c
string s("hello, world");
char *str = s;  // error, can't initialize a char* from a string
const char *str = s.c_str();  // ok
```

但是`string`类中提供了一个member function来返回一个`C-Style string`。具体来说，`c_str()`函数返回一个`指向一个null-terminated数组首元素的指针`，其中这个数组具有与`string`一样的elements。换句话说，`c_str()`提供了一个方法来获得`C-Style string`，这个返回的`null-terminated`数组与`string`中的内容一样。但是需要注意的是，`c_str()`并不能保证一直有效，任何有可能改变`s`的值的操作都会使这个返回的数组失效。