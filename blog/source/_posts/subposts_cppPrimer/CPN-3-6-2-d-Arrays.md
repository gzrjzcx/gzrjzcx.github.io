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

## Memory Dynamic Allocation for 2-d Array



## Initializing the Elements of a Multidimensional Array

