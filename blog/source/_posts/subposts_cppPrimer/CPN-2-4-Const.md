---
title: 'C++Primer 2.4: Const'
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

**注意这里的文件级实际上指的是一个编译单元。一个编译单元指的是它的`.c`文件和所有include的`.h`文件展开后的内容（因为预编译程序会将头文件在包含它的源文件中展开）。** 一个简单的实例是：

```c
/*  test.h  */
const int ci = 42;
extern const int ci_global = 100;

/*  test.c  */
#include "test.h"

/*  main.c  */
const int ci = 1;
extern const int ci_global;
cout<<ci<<endl;  // 1, ci is defined in the main.c file
cout<<ci_global<<endl;  // 42, the global const which defined in the test.h file
```



## 2. A reference to const

> A `reference to const`(const reference), which is a reference that refers to a `const type`.

注意，通常我们说`const reference`指的就是`a reference to const`。因为从本质上说，reference并不是一个object，所以reference本身是不能被const修饰的。另外，因为reference一旦初始化便不能在修改，所以从某种意义上说reference本身就是带有const性质的（reference在某种程度上可以看做是const指针）。总的来说，**const 修饰reference时其const属性是限制我们通过reference来做什么，而不是reference本身，也不是bound object**。

注意`const reference`有一个特殊情况就是reference不需要match它所引用的object的type：const reference可以被非const object或literal或expression初始化。例如：

```c
int i = 42;
const int &ri = i; // ok
const int &r2 = 42; // ok
const int &r3 = ri * 2; // ok
int &r4 = ri * 2; // error: r4 is not a const reference
```

上述const reference都能被不match的initializer初始化，但是**非**const reference则不能被const reference初始化。实际上，这种操作之所以被允许是因为`compiler将const reference绑定到了一个临时变量上`，如下代码：

```c
double f = 3.14;
const int &rf = f;
/*  transfer to:
    const int temp = f;  // floating-point is clipped
    const int &rf = temp;  // rf refers to a temp object
*/
// int &rf2 = f;  // error: compiler don't transfer the code
cout<<rf<<endl; // 3

/*  a common error  */
int &r = 0;  // error: reference must initialized to an variable
const int &r = 0;  // ok: const reference can be initialized to 0(refer to temp actually)
```

实际上，之所以能够用不匹配的类型来初始化const reference（注意只有const reference，普通的reference不行），是因为compiler实际上会创建一个未命名的临时变量来保存正在计算的表达式的中间结果。因此，一个temporary const int object `temp`被用来存储double `f`，注意此时已经发生了隐式类型转行，floating-point被截断为int；然后const reference `ri`会被绑定到`这个临时变量temp，而不是原始initializer f`。因此，此时const reference可以被不匹配类型的initializer初始化，即最终rf输出为3。但是**非const reference不会进行这种转换**，因为`此时ri所引用的object并不是我们真正想要绑定的object`。如果我们想要通过引用来修改真正的object的值，这样是达不到想要的效果的，所以此时编译器会将这种情况视为error。

关于const reference需要在此强调的一点是，一个const reference有可能引用到一个非const object。例如：

```c
int i = 42;
int &r1 = i; // r1 bound to i
const int &r2 = i; // r2 also bound to i, but cannot be used to change i
r1 = 0; // ok
r2 = 0; // error, r2 is a reference to const
```

上面的例子可以看出const reference的作用：a reference to const（此时虽然被绑定的object不是const，但是其实还是a reference to const，因为这个reference被绑定到了一个const的临时变量）只是限制了我们是否能够通过引用来修改被绑定的object。`绑定一个const reference到一个object上，并不意味着被绑定的object它本身就是const`。例如，我们不能通过`r2`来修改`i`，但是`i`还是可以被其他方法修改。

> **Conclusions**:
>
> 当我们定义了一个const reference，不是说这个reference本身是const（因为实际上reference本身不是object，不能被const修饰），也不能说明其绑定的object是const的（有可能不是，因为const reference会有可能转化为绑定到临时变量）。A reference to const只能说明通过定义了const reference，`限制了我们不能通过这个reference去修改the bound object`。



## 3. Pointers and const

A pointer to const和a pointer to reference不一样，因为指针是一个实际的object，所以`const`既可以修饰指针本身，也可以修饰指针所指向的object。一个`常量指针`（const pointer）`必须被初始化`，并且它的值（即所指向object的地址）不能被改变。`const`修饰指针的两种情况可以由下图所示：

![const_pointer](https://raw.githubusercontent.com/gzrjzcx/AlexsImageHosting/master/images/2019-11-21-22-56-58-const_pointer.jpg)

分辨两种指针的方法`就是看const修饰的是指针本身（top-leve1 const）还是修饰的是指针所指向的对象（low-level const）`。另外，因为指针本身也是一个object，所以指针赋值时不存在引用的类型不match的情况。也就是说，指针的初始化和赋值需要保持类型一致，即`const int *`和`int *`不能初始化和赋值，例如：

```c
const int i = 42;
int * const p = &i; // error: i is const but the object p refers to is not const
int const *p = &i; // ok: i is const and the object p refers to is const
/*  const int <--> int const  */
const int * const p; // ok: i is const and the object p refers to is const
// <--> int const * const p;  the pointer itself and its pointed object are both const
int *p2 = p; 
// error: p is a low-level const, its pointed object is const, but p2 points to a plain int
const int *p3 = p; // ok: p3 has the same low-level const qualification as p
```

从上面的例子还可以看到，`const int`和`int const`其实没有区别，即`const int * const p`可以看作是`int const * const p`，也就是指针本身和其所指向的对象都是const类型；`const int const *p`可以看作是`int const const *p`，实际上也还是`int const *p`，即一个`low-level const`的指针，指针本身不是const，但是指针所指向的对象是const。

> **Conclusions:**
>
> `const`可以修饰指针本身（top-level const），也可以修饰指针所指向的对象（low-level const）。指针在初始化和赋值时必须类型必须match：`top-level const`类型通常可以忽略，但是`low-level const`类型不能被忽略，必须要求等号两边的类型一致。总的来说，就是可以增加限制，不能减少限制。即`int *`可以赋值给`const int*`，但是`const int*`不能赋值给`int *`。

## 4. `constexpr` and Constant Expressions

> A `constant expression`(常量表达式) is an expression whose value  cannot change and that `can be evaluated at compile time`.

例如，一个literal（e.g. 1，'c'，3.14）是一个constant expression，一个被constant expression初始化的`const` object也是一个constant expression， 例如：

```c
const int max_files = 20; // max_files is a constant expression
const int limit = max_files + 1; // limit is a constant expression
int staff_size = 27; // staff_size is not a constant expression(not const)
const int sz = get_size(); // sz is not a constant(can't evaluate value at compile time)
```

**`constexpr`关键字用于确保一个变量是constant expression。**其定义的变量具备两个性质：

- 是`const`类型；
- 必须被`constant expression`初始化。

因为其是const类型且必须被constant expression初始化，所以保证了这个变量也是constant expression。可以被`constexpr`修饰的变量被称为`literal types`，常见的`literal types`有算数类型（arithmetic types），引用（reference）和指针（pointer）。注意`constexpr`在修饰指针的时候，**其只能修饰指针本身。**即`constexpr`修饰的指针都是`top-level const`的指针。例如：

```c
/*  global variable  */
int gi = 42;
const int con_gi = 1;

int main()
{
  const int ci = 100;
  constexpr int *pci = &ci; // error: ci is a local variable, cannot be determined at compile stage
	constexpr int *pgi = &gi; // ok: gi is a global variable; pgi is top-level const
  constexpr int *pcon_gi = &con_gi; 
  // error: pcon_gi is top-level const, but the con_gi is const
  constexpr const int *pcon_gi = &con_gi; // ok: pcon_gi is both top-level and low-level
  
  /*  a special case for reference  */
  constexpr int &rgi = gi;
  rgi = 1000; // ok
  constexpr const int &con_rgi = gi;
  con_rgi = 1000; // error
}
```

可以看到，constexpr修饰的指针都是`top-level const`的指针，即constexpr都只是修饰指针本身，让指针本身成为const类型。如果需要一个`low-level const`的指针，则还需要明确加上`const`关键字来修饰指针所指向的对象。另外，对于引用，因为引用本身不是object，所以constexpr对于引用其本身`const`的特性则会失效。例如，还是可以通过constexpr类型的引用`rgi`来修改其所绑定的对象`gi`。总的来说，`constexpr`修饰compound types，只能修饰compound types本身，不能修饰其指向的值。详细来说，就是`constexpr`只能修饰指针本身为const，以及引用本身。但是又因为引用本身不是object，不存在const类型，所以constexpr修饰的引用只是限定了所引用的值必须为constant expression。所以，constexpr并不能代替const关键字。

> **Conclusions:**
>
> `constexpr`是一个比`const`约束性更强的关键字，使用`constexpr`定义变量时不仅指明了变量为const，还要求其在编译期间就能知道值。但是`constexpr`不能代替`const`，因为对于指针，`constexpr`只能指明`top-level const`类型的指针；对于引用，`constexpr`只能指明一般类型（非const）的引用。