---
title: 'C++Primer 2.5: Dealing with Types'
date: 2019-11-23 12:57:03
categories: cppPrimer
tags: cpp
---

# Dealing with Types

随着我们的程序变得越来越复杂，各种types的名字变得越来越复杂。有些types变得越来越难拼写，有些types的含义变得越来越难以理解（或歧义）。因此，我们需要一些方法来让简化types的名字。

## 1. Type Aliases

> A `type alias` is a name that is a synonym(同义词) for another type.

Type alias实际上就是给一个类型取别名（alias）。使用shell的人应该熟悉别名的意思，比如给一个复杂的command取一个简单的别名，方便下次键入。同时，给type取别名还能让我们更加突出这个type的意义。

### `typedef`关键字

一个传统的取别名的方法是通过`typedef`关键字。与定义变量类似，`typedef`关键字定义一个`type alias`，但是要注意此时declarators（即`*`和`&`）指的是这个别名本身是什么type。例如：

```c
	typedef int price;  // price is an alias of int
	typedef price cost, *pointer;  
	// cost is an alias of cost, pointer is an alias of price *(i.e. int *);
	price b1 = 100;  // b1 is int type
	pointer p = &b1;  // p is int * type
	cout<<*p<<endl;
```

### `using`关键字

C++11提供了一种通过`using`关键字的新方法定义type alias：

```c
using price = int;  // price is an alias of int
```

个人感觉虽然`typedef`是从C就开始使用的方法，但是新的关键字`using`使用起来更加直观。另外，`typedef`看上去用宏`#define`也可以实现类似效果，但是这两个并不一样，例如：

```c
#define m_price int *

int main(int argc, char* argv[]){
	int i = 42;
	m_price pi1 = &i, pi2 = i; 
  // --> `int * pi1, pi2;`  pi1 is a pointer, pi2 is an int
	cout<<*pi1<<" "<<pi2<<endl;

	typedef int *t_price;
	t_price tpi1 = &i, tpi2 = &i; // both tpi1 and tpi2 are pointers
	cout<<*tpi1<<' '<<*tpi2<<endl;
}
```

可以看到，`#define`只是简单的文本替换，所以`pi1`是指针，但是`pi2`却是int类型。但是`typedef`则是定义了一个类型的别名，`并不是简单的文本替换`，所以使用别名`t_price`定义的变量`tpi1`和`tpi2`都是指针。同时，因为`typedef`不是简单的文本替换，所以当其定义了一个指针变量的别名且和`const`关键字搭配的时候得到的是一个`top-level const`的指针，即指针本身是const类型。例如：

```c
	typedef int *pointer;
	const pointer p = nullptr;  // p is a constant pointer
	pointer const p1 = nullptr;  // same as above
	int i = 42;
	p = &i;  // error: p is a top-level const pointer

	typedef int &ref;
	const ref r = i;
	r = 1;  // ok: can change i through r
	cout<<r<<endl; // 1
```

注意，**`typedef`不是简单的文本替换**，所以不能将`p`读成：~~`const int * p`~~。`p`实际上是一个`top-level const`类型的指针，即指针本身是const。因为`typedef int *pointer`指的是定义了一个`int`类型的别名`pointer`，**并且这个别名`pointer`是指针，即可以说`pointer是int型指针`**。所以`const`在修饰`pointer`的时候实际上修饰的是`int型指针`，所以`p`是一个const pointer而不是指向const的object。通俗地说，就是使用`typedef`定义的新类型`*pointer`会被看作是一个整体，当`const`修饰`pointer`的时候即是修饰这个整体（但不是文本替换），即修饰这个指针本身。

另外，对于引用，因为引用本身不是object，所以`const`虽然修饰的也是`&ref`整体（即引用本身），但是引用本身并不存在`const`的context。所以，最终得到的是一个普通的引用（非const引用），这也是为什么虽然定义了`const ref r = i`，但是还是可以通过`r`来修改`i`。这个情况和`constexpr`关键字一样，详情可以查看[here](https://www.hellscript.cc/2019/11/19/subposts_cppPrimer/CPN-2-4-Const/)。

> **Conclusions:**
>
> `typedef`不是简单的文本替换，在对type取别名时会将declarator一起看作是一个整体类型，所以`const`是修饰这个整体别名而不是将其拆开单独修饰。

## 2. `auto` Type Specifier

> `auto` tells the compiler to deduce the type from the `initializer`。

因为`auto`从它的`initializer`推断数据类型，所以这要求使用`auto`定义变量的时候必须初始化（有`initializer`）。在一个declaration中也可以使用`auto`定义多个变量，但这要求所有的变量的`initializer`必须是同一种type。换句话说，在一个declaration中，`auto`只能被推断成一种type。例如：

```c
auto i = 0, *p = &i;  // ok: i is int and p is a pointer to int
auto i = 42, f = 3.14; 
// error: "auto" type is "double" for `f` entity, but was previously implied to be "int"
```

对于复合类型（compound types）和`const`类型的推断，`auto`有可能被推断为与`initializer`不一致的类型。因为compiler会按照常规的初始化规则来调整推断的类型：

- `auto`对于引用类型会被推断为其所绑定的object的类型。
- `auto`对于`const`会忽略掉`top-level const`，即自身`const`的initializer会被推断为普通的base type。

例如：

```c
int i = 42;
const int ci = i, &cri = ci;
auto b = ci; // b is an int, top-level const in ci is dropped
auto c = cri; // cri is an int, top-level const in ci(bound object) is dropped
auto d = &i; // d is an int* (a pointer)
auto e = &ci; // e is an const int* (a const pointer), low-level const is kept
/*  specify top-level const or reference explicitly  */
const auto f = ci; // f is a const int
auto &g = ci; // g is a const int &(top-level is kept)
```

上例中需要注意的是当我们明确要求`auto`推断为引用类型时，`top-level const`是不会被忽略的。也就是说`g`是`const int &`，因为其绑定的对象`ci`是`top-level const`类型。另外，关于`const reference`，其指的是不能通过`reference`去修改被绑定的值，而不是说被绑定的object是const类型，具体可以看[here](https://www.hellscript.cc/2019/11/19/subposts_cppPrimer/CPN-2-4-Const/)。

## 3. `decltype` Type Specifier

因为`auto`只能从initializer中推断类型，但是有时候我们希望compiler能够为我们从一个非initializer的表达式中推断变量类型。所以C++11提供了一个新的关键字`decltype`用来从expression中推断类型，但是**`不执行这个表达式`**。例如：

```c
decltype(f()) sum = x; // sum has whatever type f returns
```

`sum`的类型将会是`f()`的返回值的类型。**`特别需要注意此时并不会执行f()，只是从其返回值的类型推断类型`**。`decltype`关键字并不会忽略`top-level const`和`reference`，例如：

```c
const int ci = 42, &rci = ci;
decltype(ci) x = 0; // x is const int, top-level is kept
decltype(rci) y = x; // y is const int &, is bound to x
decltype(rci) z; // error: z is const int &, must be initialized
```

> **Tips:**
>
> 在当前C++11标准中，`reference`只有在`decltype`的context中被当做是reference本身，其余情况下一个引用都会被当做是其所引用的对象的别名来对待。

当用一个expression让`decltype`进行类型推断时，其有可能推断出表达式结果的类型，也有可能推断出一个`reference`类型。一般来说，**如果expression产生的结果能够放在赋值操作的左边**，那么`decltype`将得到一个引用类型。例如：

```c
int i = 42, *p = &i, &r = i;
decltype(r + 0) b; // b is an int because expression yields an int result
decltype(*p) c; 
// error: c is int & and must be initilized, because derefence p get an object can be assigned
```

对于`b`，虽然`r`是一个引用，但是这是一个expression `(r+0)`，最终得到的结果是`(42+0)`即`42`，是一个`int`类型，所以此时`b`被推断为`int`类型。因为在`decltype`语境中引用并不等于被绑定的object的别名，所以如果我们想要通过`decltype`得到一个引用所绑定的对象的类型，我们可以将其作为表达式`(r+0)`来推断类型。

对于`*p`，因为`p`指针dereference的结果为`i`，是一个可以被赋值的object（即可以出现在赋值操作`=`左边的object），所以此时`decltype`推断得到的结果是一个引用，即`c`是一个`int &`类型。

另外，我们也可以强制`decltype`产生一个`reference`类型的推断：

```c
decltype((i)) d; // error: d is int & and must be initialized
decltype(i) e;  // ok: e is an int
int a, b = 2;
decltype(a = b) f; // error: f is int & and must be initialized
```

因为对于变量`i`，如果我们用括号将其括起来即`(i)`，那么compiler会将其视为expression，即这个expression `(i)`产生的结果是一个可以被赋值的object（变量`i`可以被赋值），所以`decltype`会将其推断为引用类型。同理，在赋值表达式`a=b`中产生的结果为`a`，是一个可以被赋值的object，所以`f`是一个引用类型。**`但是一定要注意此时a=b的操作并没有被执行`**，即`a`还是原来的值而不是`b`的值。

> **Conclusions:**
>
> - `auto`从initializer中推断数据类型，`decltype`从非initializer的表达式中推断数据类型。
> - `auto`会自动忽略`top-level const`，`decltype`不会。
> - `auto`会将引用类型推断为其绑定对象的类型，`decltype`会将引用类型推断为引用类型（想推断为其绑定对象类型可以用表达式`(r+0)`）。
> - `decltype`推断为引用类型还是expression结果本身的类型取决于这个expression的结果是不是可以被赋值的对象，且**`必须记住这个表达式实际上没有被执行`**。