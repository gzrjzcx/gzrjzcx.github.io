---
title: 'C++Primer 3-2: Characters in String'
date: 2019-11-25 12:32:38
categories: cppPrimer
tags: cpp
---

# Dealing with the characters in a `string`

根据C语言的使用习惯，我们通常会对一个字符串`char *`中的每个单独的字符进行处理。在C++中也是如此，我们会对字符串`string`中的每个单独的character进行处理。例如，`string`中重载的操作符`==`即是对两个`string`的每个单独的字符进行比较，看是否每个字符都相等。

C++中提供了一些库函数来对字符串中每个字符进行常见的处理：

| Function         | Explanation                                                  |
| ---------------- | :----------------------------------------------------------- |
| isalnum(`c`)     | true if `c` is a `letter` or a `digit`. (isAlphaNumber)      |
| isalpha(`c`)     | true if `c` is a `letter`.                                   |
| isdigit(`c`)     | true if `c` is a `digit`.                                    |
| iscntrl(`c`)     | true if `c` is a `control character`. (Ascii code < 32)      |
| isgraph(`c`)     | true if `c` is `printable` but not a space. (Ascii code > 32) |
| isprint(`c`)     | true if `c` is `printable` including a space. (Ascii code >= 32) |
| ispunct(`c`)     | true if `c` is a `punctuation character`. i.e. not a control character, a digit, a letter or a space. e.g. `','`. |
| isspace(`c`)     | true if `c` is a `whitespace`(i.e. a space, tab, return, newline or formfeed). |
| isxdigit(`c`)    | true if `c` is a `hexadecimal` digit.                        |
| islower(`c`)     | true if `c` is a `lowercase` letter.                         |
| isupper(`c`)     | true if `c` is a `uppercase` letter.                         |
| **to**lower(`c`) | if `c` is an `uppercase` letter, returns its `lowercase` equivalent; otherwise returns `c` unchanged. |
| **to**upper(`c`) | if `c` is an `lowercase` letter, returns its `uppercase` equivalent; otherwise returns `c` unchanged. |

在C++中上述方法都被定义在`cctype`头文件里。需要注意的是上述`whitespace`指的是`space`，`tab`，`return`，`newline`等几个特殊字符。但是`space`则只指的是**空白符**（ascii code = 32）。

> **Tips:**
>
> 在C++中，以前的C语言的头文件`[name].h`被重新命名为`c[name]`格式，来**指明这些头文件继承自C语言**。例如`cctype`则是继承自C语言库的`ctype.h`的头文件，其内容一样，但是其形式会更加适合C++的使用。因此，C++程序员应该使用`c[name]`格式的头文件。
>
> 实际上，`c[name]`格式的头文件被定义在了`std` namespace中，但是`[name].h`格式的头文件并没有。

## `Range-based for`

当我们需要单独处理`string`中每一个 character时，C++11提供了一个新方法：`range-based for`，例如：

```c
/*  syntactic form  */
for(declaration : expression)
  statement

string s("hello");
for(auto c : s)
  cout<<c<<endl;

for(auto &c : s)
  c = toupper(c);  // c is a reference and refers to the underlying element of s each time
```

其一般形式如上，`expression`是一个具有序列形式的object；然后在`declaration`中定义一个`loop control variable`，并用这个variable来获得expression中每一个element。**`注意这个loop variable每次获得的只是element的copy`**。所以如果想要通过这个`loop variable`来改变element本身，则需要**`将loop variable定义为element的引用`**，通过引用来修改element。

因此，`range for`可以被解释为：for each iteration, `c` referst to the next character in `s`, then do the statement. 即在每次循环开始的时候，`c`先被初始化为`s`的下一个字符（拷贝或引用），然后执行loop body的statement。

## `Subscript` to access individual characters

因为`range for`只适用于遍历字符串中所有字符的情况，所以当我们需要只处理字符串中部分字符的时候，需要用`subscrpt`或者`iterator`两种方法。

我们可以通过`subscript operator '[]'`来实现下标操作。对于`string`类型，`'[]'`操作符接受一个`int`（或者能够隐式转换为`int`）类型的值然后返回`string`中这个位置的character的`reference`。注意**`'[]'操作返回相应位置的字符的引用`**，也就是说，可以直接通过下标操作得到的这个引用来修改原字符的值。一个`subscript`操作遍历字符串的例子：

```c
for(decltype(s.size()) i=0; i!=s.size(); ++i)
  s[i] = toupper(s[i]);
```

注意书中使用`decltype`来推断索引`i`的值，这样可以保证`i`不会小于0，因为`string.size()`是一个`unsigned`的值。但是要注意这个`unsigned`的值如果是逆序遍历，则有可能最终出现`i = -1`然后被强制转换为最大值导致循环无法退出的情况，具体可以查看[错误示例1](https://www.hellscript.cc/2019/11/08/subposts_cppPrimer/CPN-2-1-Basic-Types/)。实际上我个人并不觉得使用`unsigned`更安全，因为虽然从定义上来说`unsigned`是不会小于0，但是实际上`unsigned`是可以被赋值为负数然后隐式转换为正数，反而会造成错误示例那样的问题。或许使用`unsigned`有其它考虑也可以留言告诉我。

> **Conclusions:**
>
> - `subscript '[]'`操作返回的是对应element的引用，因此可以直接通过这个引用来修改这个element。
> - `subscript`从下标`0`开始，即第一个字符对应的位置为`0`。
> - `subscript`需要保证在`string`中这个位置有值。换句话说，`index`需要满足条件: ` 0 <= index < size()`。如果下标操作取到不存在的值，则会造成`buffer-overflow`错误。同样，对空字符串进行下标操作也会造成这个错误。