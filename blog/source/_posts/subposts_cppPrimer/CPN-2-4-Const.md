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

> Conclusion:
>
> 当我们定义了一个const reference，不是说这个reference本身是const（因为实际上reference本身不是object，不能被const修饰），也不能说明其绑定的object是const的（有可能不是，因为const reference会有可能转化为绑定到临时变量）。A reference to const只能说明通过定义了const reference，`限制了我们不能通过这个reference去修改the bound object`。



## 3. A pointer to const

