---
title: 'C++Primer 3.5: Array'
date: 2019-12-08 12:40:29
categories: cppPrimer
tags: cpp
---

# Arrays

数组（array）是和`vector`类似的一种数据结构，也是用于存储一系列同类型的objects。但是相比于`vector`，`array offers a different trade-off between performance and flexibility`。也就是说，`vector`是**可变长度**的容器，其更注重于`flexibility`；`array`是**固定长度**的容器，对于一些固定场合其更拥有更好地`performance`。

> **Tips:**
>
> 当我们无法在编译时就确定数组的长度时，最好使用`vector`。

因为数组是固定长度的容器，所以数组在定义时其`dimension`必须是在编译期间就可知，即`dimension`必须是一个`constant expression`，例如：

```c
unsigned cnt = 42;
constexpr unsigned sz = 42;
int a[10];  // an array of ten int elements
int *pa[sz];  // an array of 42 pointers elements, each points to an int
int a2[cnt];  // error, cnt is not a constant expression
string s[get_size()];  // ok if get_size() is constexpr, error otherwise.
```

上例中需要注意的是`sz`是`constexpr`，因此其可以用来作为数组初始化的`dimension`；`cnt`则不是`constant expression`，因此编译会报错。默认情况下，数组里的`elements`会被默认初始化：全局的数组会被默认初始化为0，局部的数组会被默认初始化为`undefined values`。 另外，和`vector`一样，数组也是`objects`的容器，所以数组里的`elements` 不可能是`reference`，因为引用不是object。

## `array` intialization 

```c
int a[3] = {0,1,2};  // an array of three int elemetns with value 0,1,2
int a2[] = {0,1,2};  // an array of three int elements with value 0,1,2
int a3[5] = {0,1,2};  // an array of 5 int elements, the first three elements have value 0,1,2; the last two elements have default value
string a4[3] = {"hello", "world"};  // an array of 3 string elements, the first two elements have value "hello" and "world", the last one is an empty string.
int a5[2] = {0,1,2};  // error, too many initializers
int a6[10] = {0};  // all elements have value of 0
int a7[10] = {1};  // the first elements has value 1, others are 0

char s1[] = {'c', '+', '+'};  // list initialization, no '\0', length = 3
char s2[] = {'c', '+', '+', '\0'};  // explicit '\0', length = 4
char s3[] = "c++";  // string literals initializer, '\0' added automatically, length = 4;
char s4[6] = "Daniel";  // error, no space for null '\0', "Daniel" can be viewed as a list initializer {'D', 'a', 'n', 'i', 'e', 'l'}
char s5 = s3; // error, initialization with "{...}" expected for aggregate object
char s6[4];
s6 = s3;  // error, s6 is a constant

int a[] = {0,1,2};  // array of three ints
int a2[] = a;  // error, cannot initialize an array with another
a2 = a;  // error, cannot assign one array to another

int *ptra[10];  // ptra is an array of ten pointers elements to int
int (*pa)[10];  // pa is a pointer to an array of ten int elements
int &refa[10];  // error, no arrays of references
int (&ra)[10];  // ok, ra is a reference, binds to an array of ten ints
int *(&a)[10];  // ok, a is a reference to an array of ten pointers

```

上述例子说明了几种数组的初始化情况，首先是数组的`list initializer`。当使用`list initializer`对数组进行初始化时，可以省略掉`dimension`，此时compiler会根据`list initializer`的elements数量来推断数组的size，这样会出现三种情况（假设`list initializer`中有`n`个elements）：

- 如果`dimension == n`，那么数组中每个element的值等于`list initializer`中每个element对应的值；
- 如果`dimension > n`，那么数组中前`n`个elements的值等于`list initializer`中每个element对应的值，**数组中剩下的elements（即position大于`n`的elements）都被置为0**。
- 如果`dimension < n`，那么编译错误（C++中，C中是可能编译通过的）。 

> **Tips:**
>
> 可以使用`int a[100] = {0}`的形式将局部数组快速初始化为0。
> 因为对于`list initializer`，其会将`list`中与`array`对应的element进行赋值（即第`a`中的第一个element被`list`中的第一个元素初始化为0），**并且将后续的element的值置为0**（`a`中的其余elements被默认初始化为0）。但是这不能将数组所有elements都初始化为非0值，例如上例中的`a7[10]`，只有第一个element为1，其余为0。实际上，可以直接使用`int a[100] = {}`初始化所有元素为0，即list中不提供任何element，所有元素都会被默认初始化为0。

另外，还需要注意的就是对于`char`类型的数组，除了`list initializer`，还可以直接使用`string literal`进行初始化。但是需要注意的是**`list initializer`初始化时不会在末尾加上结束符`'\0'`，`string literal`初始化时则会自动在末尾加上结束符`'\0'`**。这是因为`string literal`作为`initializer`时会被转换为以`null`结尾的`character array`。 

同时，**一个数组不能被另一个数组初始化**。例如`s5`不能被`s3`初始化，因为`initializer`必须要求是一个`aggregate object`。并且，**一个数组也不能被重新赋值**，因为`array`本身是一个常量，数组名可以看作是一个`char const *`的指向首元素的指针。因此，一旦数组被初始化过后，便不能在被赋值。这也是为什么已经被定义过的`array`不能再次被`string literal`赋值。

最后，不存在引用数组，即数组中的element不能是`reference`，因为`reference`本身不是object，但是数组是盛放object的容器。对于`int *(&a)[10]`这类比较复杂的定义，通常解析的顺序为`由里到外，由右到左`。例如，此时`a`首先是一个`reference`，这个引用refer to一个size=10的数组，这个数组有10个pointer elements，每个指针指向一个int类型。

## Accessing the elements of an `array`

对于`built-in array`，我们有三种方法来处理数组内部的elements：`range-for`和`subscript`和`pointer（iterator）`。其中`range-for`适用于从头到尾遍历数组内的所有元素，而不需要我们自己管理`index`（因为`dimension`本身就是数组的一部分，所有系统知道数组的`dimension`）；`subscript`则和`vector`的`subscript`类似，可以实现`random access`。

为了说明如何使用`pointer`来遍历数组，我们需要先说明指针和数组的关系。在C++中，指针（pointers）和数组（arrays）是很容易混淆的概念。实际上，在大部分情况下，compiler会**将数组object转换为一个指向数组第一个元素的`Top-level`指针**。例如，如果使用`auto`对数组进行推断：

```c
int a[3] = {0,1,2};
auto pa(a); // pa is an int* that points to the first element in a
// equivalent to :
auto pa(&a[0]);
decltype(a) pa2; // pa2 is an array of ten ints
```

如上例，当我们使用数组名`a`时，compiler往往会将其替换成`&a[0]`，即指向第一个元素的指针。因此，`auto`推断得到的结果是一个指针。但是这里需要说明的是使用`decltype`对数组进行类型推断时得到的还是一个`array`，这是`decltype`的特性，类似于对`reference`推断也会得到一个`reference`。

知道了数组名在使用时会被转化为指针后，我们可以利用指针作为迭代器来遍历数组。需要做的就是得到一个指向数组第一个元素的`pointer`（类似于`begin()`返回的`iterator`）和指向最后一个元素的下一个元素的`pointer`（类似于`end()`返回的`iterator`）：

```c
int a[10] = {0,1,2,3,4,5,6,7,8,9};
int *b = a;  // b points to the first element in a
int *e = &a[10];  // e points to the element which past the last element in a
for(int *p = b; p!=e; ++p)
  cout<< *p <<endl;
```

上例中，其实与`vector`中的迭代器遍历方法类似，`b`通过数组名会被转化为指向数组内第一个元素的指针的特性得到一个指向数组第一个元素的指针；`e`通过指针与整形的运算得到一个指向最后一个元素的下一个元素的指针（`off-the-end pointer`）。至此，我们实现了通过指针（迭代器）遍历数组的方法。需要注意的是`off-the-end pointer`实际上并不指向数组中任何`element`，因此**不能对`off-the-end pointer`进行dereference操作和increment等操作**。只能通过`address-of operation '&'`对其进行取地址操作。

另外，尽管上述方法可以得到指向首元素的指针和`off-the-end`指针，但是并不是很直观。C++11在`iterator.h`中提供了两个方法来更方便地获取两个所需的指针：

```c
int a[10] = {0,1,2,3,4,5,6,7,8,9};
int *b = begin(a);  // takes argument 
int *e = end(a);  // takes argument
for(int *p=b; p!=e; ++p)
  cout<< *p <<endl;
```

这两个方法并不是`container`里的成员函数，而是定义在`iterator.h`里的方法。因此注意这两个方法需要传入这个数组作为参数。

利用指针作为`iterator`来遍历数组的方法依赖于指针的算术意义。实际上，在[iterator](https://www.hellscript.cc/2019/11/29/subposts_cppPrimer/CPN-3-4-Iterator/)中我们介绍了C风格的指针实际上是一种`random access iterator`，因此在那篇文章中介绍的`iterator arithmetic`同样适用于`pointer arithmetic`。例如，**在同一个数组中**比较两个指针的大小实际上是比较两个指针所指向的element在数组中的位置的大小。

另外还有一点需要注意的是，与`vector`和`string`等`library types`不同的是，`built-in array`的`subscript`操作是C++语言自身的一部分，而不是在类里重载的操作符。因此，`array`的`subscript`操作并不要求`index`是`unsigned value`。但是，其同样要求`subscript`的结果必须指向数组里的一个element（或者`one past the end of`）。

```c 
int a[10] = {0,1,2,3,4,5,6,7,8,9};
int i = a[2];  
/*  equivalent to :
		int *p = a;
		i = *(p + 2);  */

int *p = &a[2];  // p points to the element indexed by 2
int j = p[1];  // equivalent to *(p+1), p points to the element a[3]
int k = p[-2];  // p[-2] is the element a[0]
```

上例中对于数组的取下标操作，实际上可以看作两步：

- 先取得指向数组首元素的指针`p`；
- 对指针`p`进行指针运算，取到相应位置上的`element`。

因为内置数组中C++并不规定`index`必须要是`unsigned`，所以其也可以是负数。即`p[-2]`表示的是`p`所指向的`element`的position往前两个元素。

## `array` VS. `pointer`

在C/C++中，数组和指针是两个非常容易混淆的概念，尤其是`数组名`和`指针`。 实际上，数组是数组，指针是指针，两者是不同的概念。

```c
int a[3];  // an array
a++;  // error
int *p;  // a pointer
p++;  // ok
p = a;  // p points to the first element in a
```

当定义一个数组`int a[3]`时，数组名`a`是数组的首地址，**是一个地址，在汇编层面上看是一个常量。**因为大部分compilers在碰到数组定义的时候都会将其相关信息（`dimension`，`type`，`initial_addr` etc.）存储在符号表中。也就是说，数组定义后其位置就固定了，因此其**首地址在编译阶段是可以确定的**。当每次遇到数组名`a`这个标识符时，compiler就会从符号表中取出这个`initial_addr`来代替数组名`a`。例如`a`的`initial_addr = 0x01`，那么每次碰到标识符时，例如`subscript`操作：`a[1]`会被替换为`*(0x01 + 1)`。因此，可以说在汇编层面上数组名完全被转化为了一个`constant`，即固定的值。这也是为什么不能对数组名`a`进行自加自减操作。详细[here](http://www.spongeliu.com/28.html)。但是，对于指针`p`，其本身是一个变量，是一个`object`，它的值存储在`&p`的地址（即`p`本身的地址）上。**`p`的值在编译阶段是不能确定的，**所以其可以进行自加自减操作。

因为这个原因，当我们进行`subscript`操作（e.g. `a[1]`）时，只需要访问内存一次：因为`a`是常量，只需要从符号表中取出`a`的值进行`+1`的操作后，从内存中取出相应位置的element。但是`*(p+1)`则需要访问内存两次：先从内存中`&p`（i.e. `p`本身的地址）这个地址中取出`p`的值（i.e. `&a[0]`），然后根据`&a[0]`的地址从内存中取出相应位置的element。

另外，之前也说到过，在大部分情况下，`array`会被compiler转换为指向数组首元素的`top-level pointer`。但是，还是存在几种特殊情况，C11 standard (ISO/IEC 9899:2011):  6.7.6.2 Array declarators (p: 130-132)规定了：

Any [lvalue expression](https://link.zhihu.com/?target=https%3A//en.cppreference.com/w/c/language/value_category) of array type, when used in any context other than 

- as the operand of the [address-of operator](https://link.zhihu.com/?target=https%3A//en.cppreference.com/w/c/language/operator_member_access)
- as the operand of [sizeof](https://link.zhihu.com/?target=https%3A//en.cppreference.com/w/c/language/sizeof)
- as the string literal used for [array initialization](https://link.zhihu.com/?target=https%3A//en.cppreference.com/w/c/language/array_initialization)
- as the operand of [_Alignof](https://link.zhihu.com/?target=https%3A//en.cppreference.com/w/c/language/_Alignof)

undergoes an [implicit conversion](https://link.zhihu.com/?target=https%3A//en.cppreference.com/w/c/language/conversion) to the pointer to its first element. The result is not an lvalue.

也就是说，`sizeof`和`&`操作时`array`都不会被转换为指针。首先看对`array`取地址的情况：

```c
	int a[3] = {1,2,3};
	cout<<a<<endl;  // 0x7ffee674e788
	cout<<&a<<endl; // 0x7ffee674e788
	cout<<a+1<<endl;  // 0x7ffee674e78c
	cout<<&a+1<<endl;  // 0x7ffee674e794

 sizeof(a);  // 12
```

可以看到，`a`此时被转换为了指向第一个元素的指针，所以指针`a`输出的结果是数组的首地址，即第一个元素的地址；`&a`拿到的地址和`a`拿到的地址相同，但是并不是说指针`a`和指针`&a`相同！因为`a+1`得到是指向`数组内`的下一个元素（即`2`）的地址，但是`&a+1`拿到的却是指向`该数组之后`的下一个元素（`undefined`）的指针。换句话说，`a`和`&a`的**类型不同**（指针的类型决定了compiler如何解释指针，详细可以查看[here](https://www.hellscript.cc/2019/02/22/subposts_c/The-nature-of-pointer/)，`a`是指向数组第一个元素的指针（即`int *a`，其实也就是`&a[0]`），`&a`则是指向整个数组的指针（即`int (*a)[3]`），但是他们的值都相同（都是数组的首地址）。参考[here](https://blog.csdn.net/daniel_ice/article/details/6857019)。另外，`sizeof(a)`得到的却是整个数组的大小，因为此时`a`并没有被隐式转换为指针。

> **Conclusions:**
>
> - 在编译层面上`array`可以看作是一个地址**常量**，指针则是一个变量。
> - 大部分情况下`array`会被隐式转换为一个`指向该数组的首元素的指针，即&a[0]`。但是有四种情况不会（这里说明常见的两种）：
>   - 因为`array`是常量，所以取地址符`address-of '&'`对于`array`的含义为**决定指针的类型**，但是它们的值都是数组的首地址：
>     - `a`：`int*`类型，即一个普通指针，也就是`&a[0]`。
>     - `&a`：`int (*a)[]`类型，即一个数组指针，其指向整个数组。
>   - `sizeof(a)`得到的是整个数组的大小。因此此时`a`并没有被转换为指针。

