---
title: Memory Dynamic Allocation for 2-d Array
date: 2019-12-11 16:37:49
categories: C
tags: c
---

# Memory Dynamic Allocation for 2-d Array

{% colorquote info %}
因为在C++中并不存在真正的[2-d array](https://www.hellscript.cc/2019/12/10/subposts_cppPrimer/CPN-3-6-2-d-Arrays/)，所以往往通过不同的方法来模拟实现二维数组。又因为栈上的空间有限，所以实际上我们通常通过以下4种方法在堆上动态分配内存来模拟实现二维数组。
{% endcolorquote  %}

一般来说，我们主要通过不同的`操纵内存的方式`是实现二维数组。例如，我们可以通过一个`二级指针`来实现实现内存空间连续的二维数组；也可以通过一个`二级指针`来实现内存空间不连续的二维数组。这只有取决于我们`如何分配内存空间`。通常，我们有以下四种分配内存空间的方式，下面四种方法都以三行三列（`ROW = 3, COL = 3`）的二级数组为例：

## 通过`二级指针`实现`内存不连续`的二维数组：

```c
void print2d(int **a)
{
	for(int i=0; i<ROW; i++)
	{
		for(int j=0; j<COL; j++)
			printf("%p ", *(a+i)+j);
		printf("\n");
	}
}

void malloc2d_1(int **p)
{
	p = (int **)malloc(ROW * sizeof(int *));
	for(int i=0; i<ROW; i++)
		*(p+i) = (int *)malloc(COL * sizeof(int));

	print2d(p);

	for(int i=0; i<ROW; i++)
		free(*(p+i));
	free(p);	
}

int main(int argc, char* argv[]){
	int **p = NULL;
	malloc2d_1(p);
	p = NULL;
}
/*  results : 
*(p):   0x7f8f59c00630 0x7f8f59c00634 0x7f8f59c00638 
*(p+1): 0x7f8f59c00690 0x7f8f59c00694 0x7f8f59c00698 
*(p+2): 0x7f8f59c006a0 0x7f8f59c006a4 0x7f8f59c006a8  */
```

因为数组在实际使用中基本上是通过被转换为`指向首元素地址`的指针来进行`subscript`等操作，所以我们可以通过一个二级指针来模拟实现二维数组。如上例中，我们定义了一个`二级指针 p`，并且首先给它分配了`3个指针大小`的内存空间，它们的值则分别是指向第一维`row`的三个elements（因为`COL=3`）的地址。然后在用一个`for loop`，分别给`*(p), *(p+1), *(p+2)`分配`3个int大小的内存空间`，用来存储相应的`int`数据。总的来说，就是先给一个二级指针分配`row`个指针大小的空间作为分别指向每个`element（数组）`的首地址的指针，然后在分别给这每个`element（数组）`分配`col`个`int`大小的内存空间来存储相应的数据。注意此时`p`是一个二级指针，并不是数组，`sizeof(p), sizeof(*p), sizeof(*(p+1))...`结果都是8，因为他们都只是一个指针，而不是数组，**这里只是利用指针来模拟数组，并没有出现真正的`array` type**。同样的，此时如果对`p`取地址，那么得到的将是这个二级指针的地址，其值与`p`并不一样。

另外需要注意这种分配内存的方式得到的**每个“一维数组”内存地址并不是连续的**。从输出的结果可以看到，`*p`的第三个element的地址是`0x7f8f59c00638`，但是`*(p+1)`第一个element的地址则变成了`0x7f8f59c00690`。换句话说，这种方式只保证了每个一维数组中的元素是连续存储的，但是每个一维数组间并不是连续存储。这从实际分配内存的代码上也可以看到，其是通过在`for loop`里面每次都重新对每一行数组进行一次内存分配，而不是一次直接将所有的elements所需内存分配好。因此，在释放这段内存的时候也需要先用`for loop`将每一行数组先释放，最后在释放这个二级指针所指向的内存。其内存方式如图：

![IMG_0009.PNG](https://raw.githubusercontent.com/gzrjzcx/AlexsImageHosting/master/images/2019-12-15-20-26-05-IMG_0009.PNG.png)

从图中可以看到，其总共调用了4次`malloc`，每次`malloc`都会返回一个指向该段内存首地址的指针。因此，此时`memory 2`，`memory 3`和`memory 4`都**不是连续的内存空间**，这也是这种内存分配方式最大的特点。注意图中的灰色方块指的是地址（即指针的值），而不是这个指针所指向地址的值（即解引用得到的`underlying object`）。例如，`memory 2`中的指针`*(p+1)`的值是`0x7f8f59c00690`，对这个指针解引用则会得到`*(*p+1)`，即`p[1][0]`。

## 通过`二级指针`实现`内存连续`的二维数组：

```c
void print2d(int **a)
{
	for(int i=0; i<ROW; i++)
	{
		for(int j=0; j<COL; j++)
			printf("%p ", *(a+i)+j);
		printf("\n");
	}
}

void malloc2d_2(int **p)
{
	p = (int **)malloc(ROW * sizeof(int *));
	*p = (int *)malloc(ROW * COL * sizeof(int));
	for(int i=1; i<ROW; i++)
		*(p+i) = *(p+i-1) + COL;
	print2d(p);
	free(*p);
	free(p);	
}

int main(int argc, char* argv[]){
	int **p = NULL;
	malloc2d_1(p);
	p = NULL;
}
/*  results : 
*p:     0x7f8f59c02a70 0x7f8f59c02a74 0x7f8f59c02a78 
*(p+1): 0x7f8f59c02a7c 0x7f8f59c02a80 0x7f8f59c02a84 
*(p+2): 0x7f8f59c02a88 0x7f8f59c02a8c 0x7f8f59c02a90 */
```

同样是使用二级指针来模拟二维数组，与上述方法不同的是使用`malloc`一次申请了`ROW * COL`大小的内存空间，而不是像之前一样每次只申请`COL`大小的内存空间，然后申请`ROW`次，所以，这样得到的是**连续的内存空间**。但是，这种方式还需要手动将`*(p+1)`与`*(p+2)`等后续指针指向内存中相应的位置，这样才能正确得到二维数组。

![IMG_0008.PNG](https://raw.githubusercontent.com/gzrjzcx/AlexsImageHosting/master/images/2019-12-15-20-26-42-IMG_0008.PNG.png)

## 通过`一维数组`来实现`二维数组`

除了使用二级指针，我们也可以直接使用一个一维数组（或一级指针）来模拟实现二维数组。这是因为， 数组的**内存空间是连续的**，所以我们只要手动使用两个`index`即可实现二维数组。但是因为栈上的空间有限，所以通常我们会在堆上申请空间。

```c
void malloc2d_3(int *p)
{
	p = (int *)malloc(ROW * COL * sizeof(int));
	for(int i=0; i<ROW; i++)
		for(int j=0; j<COL; j++)
			printf("%p\n", p+(i*COL+j));
	free(p);
}

int main(int argc, char* argv[]){
	int *p = NULL;
	malloc2d_3(p);
	p = NULL;
}
```

这种方法比较简单，就是通过`malloc`一次性申请`ROW * COL`大小的内存空间，然后通过`i`和`j`两个`index`来拿到相应位置的element。很明显，这样的二维数组其内存空间也是连续的。

## 通过一个`数组指针`来实现`二维数组`

我们也可以采用真正的二维数组（`数组的数组`）的方式来实现动态内存分配的二维数组，即通过一个数组指针来实现。

```c
void malloc2d_4(int (*p)[ROW])
{
	p = (int (*)[ROW])malloc(ROW * COL * sizeof(int));
	for(int i=0; i<ROW; i++)
		for(int j=0; j<COL; j++)
			printf("%p\n", *(p+i)+j);
	free(p);
}

int main(int argc, char* argv[]){
	int (*p)[ROW] = NULL;
	malloc2d_4(p);
	p = NULL;
}
```

这其实和在栈上直接使用`数组的数组`的方式来实现二维数组一样，只不过现在是在堆上开辟了内存空间。因为其也是使用`malloc`一次直接申请了`ROW * COL`大小的内存空间，所以其也是内存连续的二维数组。