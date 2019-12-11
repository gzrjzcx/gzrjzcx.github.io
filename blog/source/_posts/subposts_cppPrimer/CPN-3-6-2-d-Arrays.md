---
title: 'C++Primer 3-6: 2-d Arrays'
date: 2019-12-10 12:47:52
categories: cppPrimer
tags: cpp
---

# Multi-dimensional Arrays

> 严格来说，**在C++中并不存在多维数组，当我们说多维数组时其实是在说`数组的数组`**，数组的数组指的就是一个数组，但是其element也是一个数组。

例如：

```c
int a[3][4]; // a is an array of size 3; each element is an array of size 4
int arr[10][20][30]; // arr is an array of size 10; each element is an array of size 20; each element of this 20-element array is an array of ints of size 30
```

从上述定义可以看到，当我们定义多维数组时，通常都是定义`element`也是数组的数组。我们可以从左往右理解多维数组的定义。例如`arr`，

- 首先看其是`arr[10]`，是一个有10个`element`的数组；
- 然后看`arr[10][20]`，`arr[10]`这个数组的每个`element`都是一个有20个`element`的数组；
- 最后看`arr[10][20][30]`，从`arr[10][20]`这些数组中每个`element`都是一个有30个`ints element`的数组；

对于一个二维数组，通常我们将其`first dimension`称为`row`，`second dimension`称为`column`。即从左往右看第一维是`行`，第二维是`列`。

##  `2-d array` Vs. `secondary pointer`

对于`array`，一定要明确的的一点就是**数组是数组，指针是指针，两者是不同的`data type`**。尽管在大部分情况下compiler会将`数组名`转换为`指向首元素的指针`，但是他们本质还是不同的类型。这一点对于多维数组同样适用，也就是说`二维数组`并不是`二级指针`，注意这里说的`二维数组`指的是数组的数组，**C++中只有一维数组！**。例如：

```c
int a[3][3];
cout<< a <<endl;  // 0x7ffeef5e2708, int (*)[3]
cout<< &a <<endl;  // 0x7ffeef5e2708, int (*)[3][3]
cout<< a[0] <<endl;  // 0x7ffeef5e2708, int * 
cout<< &a[0] <<endl;  // 0x7ffeef5e2708, int (*)[3]
cout<< &a[0][0] <<endl;  // 0x7ffeef5e2708, int *
```

这里首先要先深入理解对于一维数组取地址操作的结果，不懂可以先看[here](https://www.hellscript.cc/2019/12/08/subposts_cppPrimer/CPN-3-5-Array/)。我们知道大部分情况下一维数组会被编译器转换为指向数组首元素的指针，即一个普通指针`int *`；但是当我们对一维数组进行取地址时，得到的是一个指向整个数组的指针，即数组指针`(*)[]`。这也适应于多维数组。以二维数组为例，我们来分析上面例子。

首先，因为二维数组实际上是**一个其element也为数组的一维数组**，所以该二维数组的数组名`a`也会被compiler转为一个指向数组首元素的指针。但是，**因为这第一个element也是一个数组**，所以`a`被转换为一个**指向数组的指针`int (*)[3]`（也就是数组指针）**。类似于一维数组，对`a`在取地址实际上得到的指向整块内存的指针（也就是整个二维数组），所以其类型为`int (*)[3][3]`。同样地，`sizeof(a) == 36`，也就是整个二维数组的大小。

然后，对于`a[0]`，其实也就是`*(a+0)`，也就是对于`a`的解引用，得到的是`a`的第一个element，**因为这第一个element也是一个数组**，所以compiler也会将这个数组转换为指向这个数组首元素的指针。此时其实已经回到了一维数组的情况，这个指针类型为`int *`。同样地，对这个一维数组转为的指针取地址，则会得到一个`指向整个数组的指针`，也就是说`&a[0]`得到的指针是`int (*)[3]`，与`a`相同，且`sizeof(a[0]) == 12`，也就是`a`的第一个`element`对应的数组的大小。

> **Conclusions:**
>
> - 对于二维数组，其是一个`element`也是数组的一维数组。所以，`subscript`操作得到的是对应`position`的数组。
>
> - 上述几种操作得到的指针，尽管地址都一样，但是根据**类型不同**我们可以有如下关系：
>
>   - `a == &a[0]`;
>   - `a[0] == &a[0][0]`
>
>   但是`a`和`&a[0]`并不等于`a[0]`，因为它们的类型不同。

## `(*)[]` Vs. `**`

上述分析中可以看到，二维数组`a`会被转换为一个`指向数组的指针`，即数组指针。注意这个数组指针（`(*pointer)[]`）和二级指针（`**pointer`指向指针的指针）是两个不同的类型，并不能通过强制转换来混用。

- 数组指针指的是一个**指向数组的指针**，即`(*)[]`。
- 二级指针指的是一个**指向指针的指针**，即`**`。

例如，如果函数的形参是数组指针，但是传入的实参是二级指针，那么`icc`下编译的话会报错：`argument of type "int (*)[3]" is incompatible with parameter of type "int **"`。如果强制将数组指针转为二级指针，编译可以通过，但是所得到的地址是不对的，此时如果解引用的话有可能出现`segmentation fault`。例如：

```c
void print2d(int **a)  // a pointer to pointer
{
	for(int i=0; i<3; i++)
	{
		for(int j=0; j<3; j++)
			printf("%d ", *(*(a+i)+j));
		printf("\n");
	}	
}

int main(int argc, char* argv[]){
	int a[3][3] = {{1,2,3}, {4,5,6}, {7,8,9}};
	for(int i=0; i<3; i++)
	{
		for(int j=0; j<3; j++)
			printf("%p ", (*(a+i)+j));
		printf("\n");
	}
	printf("%s\n", "-------------------");

	print2d((int **)a);  // a is a pointer to array, is converted to a pointer to pointer
}

/*  results:
0x7ffeeb62e700 0x7ffeeb62e704 0x7ffeeb62e708 
0x7ffeeb62e70c 0x7ffeeb62e710 0x7ffeeb62e714 
0x7ffeeb62e718 0x7ffeeb62e71c 0x7ffeeb62e720 
-------------------
[1]    4355 segmentation fault  ./bin/leetcode */
```

上例中，如果把`void print2d(int **a)`函数改为`void print2d(int (*)[3])`，那么可以得到正确的结果。另外，这个例子也可以看出，局部的二维数组是**在栈上的一块连续的内存**。因为线程的栈上的空间有限（一般默认为1M），所以我们也可以使用`内存动态分配`的方法在堆上模拟创建二维数组。

关于数组指针和二级指针的区别，我们还可以在看一个例子：

```c
	int a[3][3] = {{1,2,3}, {4,5,6}, {7,8,9}};
	int **pp;
	int (*pa)[3] = a;  // use 2-d array a as a initializer, must specify the lower dimension
	cout<<sizeof(pp)<<endl;  // 8, a pointer to pointer
	cout<<sizeof(pa)<<endl;  // 8, a pointer to array
	cout<<sizeof(a)<<endl;   // 36, now a is a 2-d array instead of a pointer

	pp = (int **)a;  // convert (*)[3] to **
	cout<<a<<endl;  // 0x7ffeeb806690
	cout<<a+1<<endl; // 0x7ffeeb80669c
	cout<<**a<<endl;  // the first element 1
	cout<<pp<<endl;  // 0x7ffeeb806690
	cout<<pp+1<<endl;  // 0x7ffeeb806698
	cout<<**pp<<endl;  // segmentation fault
```

从上例中可以看到，不管是数组指针`int (*pa)[3]`还是二级指针`int **p`，**他们都是一个指针，**其`sizeof`都为8。但是对于`sizeof(a)`，根据之前的解释，数组在`sizeof`操作的时候并不会被转化为指向首元素的指针，所以此时得到的结果是整个数组的size。但是当我们使用`a`来初始化一个数组指针的时候，此时数组指针的`dimension`一定要与该数组的较低一个`dimension`相匹配。详细来说，`int (*pa)[] = a`在`icc`下并不能编译通过，因为`(*)[]`和`(*)[3]`也被编译器认为不是同一种类型。实际上，`(*)[3]`提供了更多的信息给编译器，它说明了指针`pa`指向的数组的类型是`int [3]`。所以，如果想要利用一个指针来access二维数组内的值，那这个指针必须是和`a`一样的类型，也就是`int (*pa)[3]`。这样就可以通过这个数组指针`pa`来获取数组内部的元素，例如`*(*(pa+1)+2)`得到的结果就是`6`。注意此时并没有使用`a`转换而来的指针来获取数组内部元素。

再看，如果我们强制将`a`转换为一个二级指针`pp`，这样虽然其`a`和`pp`的值（即所存放的地址）都是一样的（都是数组首地址），但是会导致编译器解析这段内存失败（寻址错误）。因为`a`和`pp`的值都是这个数组的首地址，所以这段内存应该按照定义这个数组`a`的方式解析，即第一个元素应该是一个`int [3]`的数组（因为是`int [3][3]`的二维数组），其地址为`0x7ffeeb806690`；第二个元素是另一个`int [3]`的数组，其地址是`0x7ffeeb80669c`...此时如果是通过数组指针`a`，即`(*)[3]`来寻址，将会是正确的解析结果。但是如果是通过被强转过来的二级指针`pp`进行寻址，其第一个元素的地址是`0x7ffeeb806690`，但是第二个元素的地址是`0x7ffeeb806698`，并不是正确的解析结果，所以会导致`segmentation fault`。这是因为指针的大小是固定的，即`sizeof(pointer) = 8`。所以通过二级指针来解析这段内存的话，第二个元素的地址会是`0x7ffeeb806690+8`。但是实际上这段内存的正确解析应该是第二个元素的地址为`0x7ffeeb806690+12`，因为`sizeof(int [3]) = 12`。

>**Conclusions:**
>
>数组指针是数组指针，二级指针是二级指针。两者是不同的东西，不能混用。即使有时候他们两保存了相同的地址，但是因为类型不同，其解析也不一样。

## Memory Dynamic Allocation for 2-d Array

我们已经知道，在C/C++中并不存在真正的二维数组，而是数组的数组。所以，其实除了使用数组的数组的方法，我们也可以通过动态分配内存的方式实现二维数组。

## Initializing the Elements of a Multidimensional Array

