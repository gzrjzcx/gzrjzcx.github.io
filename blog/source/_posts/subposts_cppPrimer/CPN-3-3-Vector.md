---
title: 'C++Primer 3.3: Vector'
date: 2019-11-25 16:00:53
categories: cppPrimer
tags: cpp
---

# Library `vector` Type

> a `vector` is a collection of **objects**, all of which have the **`same type`**.

在C++中，一个`vector`就是`一系列相同类型的objects`。通常，`vector`也被称为容器(container)，因为它就像是一个容器来盛放其它objects。

`vector`是一个类模板（class template），因此我们在使用的时候必须要提供额外的信息来创建这个类。`模板（templates）`不是方法（function）或者类（class），而可以看作是指导compiler如何产生类或函数的**指令**（instructions），或者说规则。而compiler从模板中产生类或者方法的过程叫作`实例化（instantiation）`。当我们要使用一个目标来实例化类的时候，需要指明（即提供额外信息）我们想要实例化哪一种类或方法。模板是一个比较复杂的东西，我们会在之后详细说到它。

> **Tips:**
>
> - `vector`是一个`template`，不是一个type。但是从`vector`模板中生成的包含element type的如`vector<int>`则是type。
> - 因为`reference`不是object，所以`vector`并不能用来盛放`reference`。

##  `vector` Summary

`vector`作为一种可变长度的STL容器，其底层数据结构为`数组`。因此，`vector`也具有`O(1)`时间的快速访问效率；同时，因为其是`顺序存储（连续的内存空间）`，所以插入(insert)和删除(erase)效率都较低（`O(N)`）。

相比于传统的`数组(array)`，`vector`的优势在于可以动态调整大小，其扩容规则一般以2倍扩大的方式增长。例如：

```c
	vector<int> v;
	cout<<"initial capacity = "<< v.capacity()<<endl; 
	cout<<"initial size = " << v.size()<<endl;
	cout<<"=========="<<endl;

	for(int i=0; i<5; i++)
	{
		v.push_back(i);
		cout<<"capacity = " << v.capacity()<<endl;
		cout<<"size = "<<v.size()<<endl;
		cout<<"---------"<<endl;
	}

/*  results  */

initial capacity = 0
initial size = 0
==========
capacity = 1
size = 1
---------
capacity = 2
size = 2
---------
capacity = 4  //  2 * 2 = 4
size = 3
---------
capacity = 4
size = 4
---------
capacity = 8  //  4 * 2 = 8
size = 5
---------
```

 注意上述`size()`指的是`vector`中elements的数量，`capacity`则指的是`vector`的容量。在新建一个`vector`的时候，其初始大小`capacity = 0`，随着第一个元素`0`被push进去，其大小扩充为`capacity = 1`；随着第二个元素被push进去，其大小扩充为`capacity = 2`，随着第三个元素被push进去，其大小扩充为`capacity = 2*2 = 4`... 另外，需要注意的是其具体的扩充方式为：

- 首先重新申请一个2倍大小的`连续内存空间` ；
- 然后将原内存空间的内容`copy`过来；
- 最后将原空间内容进行释放，将内存空间交还给OS。

这也是为什么如果`vector`发生了扩容，那么所有的迭代器的都会失效的原因。

另外，对于一般的`vector`初始化:

```c
vector<int> v;
cout<<sizeof(v)<<endl;  // 24

/*  data member of vector  */
iterator start;  // points to the start of the used memory
iterator finish;  //  points to the end of the used memory
iterator end_of_storage;  // points to the end of the whole allocated continuous memory
```

其`sizeof`得到的结果是其自身的大小。因为`sizeof`得到的是object本身的大小，而`vector<int>`本身是`vector`这个`class template`的实例化，所以`sizeof(v)`得到的是这个类`vector<int>`创建的对象`v`本身的大小，其与compiler有关。可以看到`vector`一般有3个成员变量，每个大小为8，所以在我这icc下`sizeof`为24。

另外，**`vector`中的elements会被存放到堆（heap）上**，也就是说不管其中有多少个elements，`sizeof`的结果不会改变，具体[here](https://blog.csdn.net/theonegis/article/details/50533770)。即使element在栈（stack）上被创建，在`v.push_back()`的时候会调用object的copy constructor将element拷贝到堆上进行存储，并且`vector`内部通过指针来访问这个object的地址，具体[here](https://cloud.tencent.com/developer/article/1386030)。

也就是说，`vector`中其实保存的并不是`object`本身，而是在`push_back()`时通过`object`自身的`copy constructor`将这个object拷贝到堆上（即`std::move`）`finish`迭代器（此时可以理解为指针）所指向的内存（也即是`vector`的`capacity`大小的连续内存中的一块）。这个过程涉及到了`vector`的底层实现的源码，在此不展开，有兴趣可以查看[here](https://blog.csdn.net/cxc576502021/article/details/83020617)。

> **Conclusions:**
>
> 从宏观上来说，`vector`内部维护了三个iterator（此时可看作指针），来控制堆上的一块连续内存。elements被存储在堆上的这块连续内存上，vector本身则存储在栈上（默认情况下）。说objecs被存放在`vector`中只是一种从作用和效果上考虑的说法，实际上vector中并没有存放objects。
>
> 所以，`vector`可以看作是通过几个迭代器（封装好就是`vector`）来操纵一个堆上的数组，只不过通常这几个迭代器在栈上，这个数组在堆上。

## `vector` Definition and Initializition

与其他class type一样，`vector`有很多种初始化方法，同时C++11还提供了`list initializer`：

```c
vector<int> iv;  // definition, iv is an empty vector
vector<int> iv2(iv);  // copy elements of iv into iv2
vector<int> iv2 = iv;  // equivalent to the above
vector<string> sv(iv);  // error: sv holds strings not int;

/*  parentheses initializer (count, value)  */
vector<int> iv3(10, -1);  // ten int elements, each initialized to -1
vector<int> iv4(10);  // ten elements, each initialized to 0
vector<string> sv2(10, "hi!"); // ten string elements, each initialized to "hi!"
vector<string> sv3(10); // ten string elements, each initialized to empty

/*  list initializer  */
vector<string> sv4{"a", "an", "the"};
// three elements, the first holds "a", the second holds "an", the third holds "the". Must use curly braces.
vector<string> sv4("a", "an", "the");  // error: must use {}

/*  mixed curly braces initializer {} and parentheses initializer ()  */
vector<int> iv5(10);  // ten elements with value 0
vector<int> iv6{10};  // one element with value 10
vector<int> iv7(10, 1);  // ten elements with value 1
vector<int> iv8{10, 1};  // two elements with value 10 and 1
vector<string> sv5{"hi"};  // list initialization, one element
vector<string> sv5("hi");  // error: can't construct a vector from a string literal
vector<string> sv6{10};  // ten elements with empty string
vector<string> sv7{10, "hi"}  // ten elements with value "hi"
```

首先，如果使用一个`vector`来初始化另一个`vector`，不管是`copy initialization`还是`direct initialization`，其含义都是**拷贝initializer中所有的elements到新创建的这个vector中**。注意新vector中的每一个元素都是initializer中对应的每一个元素的`copy`，而不是`reference`。

因为C++11中新增了`list initializer`，即使用curly braces`{...}`来作为initializer，所以要区分其与`direct initializer`（即使用parenthesis`()`）的区别。尤其是当initializer和vector的类型不一致的时候。例如，`sv5`，`sv6`和`sv7`都使用了`{}`，但是只有`sv5`是`list initialization`。**因为并不能使用`10`作为initializer value来初始化`sv6`，所以此时compiler会尝试使用`parenthesis initializer`的方法来初始话这个vector。**

> **Conclusions:**
>
> - `list initializer`表示的是使用`{}`中的每一个value来进行初始化，**`{}`中有几个值则vector中有几个elements，每个element的值是`{}`中对应的value。**即`list initializer`一次可以提供多个初始值，`{}`中每个element都是一个初始值，但是`parenthesis`只能提供一个初始值。
> - `parenthesis`表示的是使用`in-class initializer`即`()`作为initializer。`()`中有两个参数：`(count, value)`。
> - 当使用`list initializer`来初始化，但是`{}`中的值并不能初始化vector时，即`vector<type>`中的`type`与`{}`中的值的type不一致时（例如`sv6`），compiler则会考虑使用其他初始化方法。这种情况只针对`list initializer`，对于`parenthesis`则不会，类型不对时则会直接报错。

实际上，尽管我们可以在`vector`创建时通过初始化直接给`vector`添加元素并赋值，但是`vector`更多地适用于数组大小未知的情况。因此，大部分情况下我们都会定义一个空的`vector`然后通过`push_back()`来给这个`vector`添加元素。事实上，因为C++标准对于`vector`的implementation有效率要求，所以在创建`vector`时如果指定了`vector`的大小反而有可能对效率造成影响（除非所有的elements都是一样的值）。在C中，一般对于不知道数组大小的情况，我们通常通过单链表来解决；在C++中则可以很方便地使用`vector`。

## `vector` Common Operations

| Functions          | Explanation                                                  |
| ------------------ | ------------------------------------------------------------ |
| `v`.empty()        | Returns `true`if `v` is empty; otherwise returns false.      |
| `v`.size()         | Returns the `number of elements` of `v`.                     |
| `v`.capacity()     | Returns the **current** maximum `available memory space` for elements `(expressed in terms of elements)`. |
| `v`.push_back(`t`) | Adds an element with value `t` to end of `v`.                |
| `v.`pop_back()     | Removes the last element in the vector, effectively `reducing the container size by one`. |
| `v`[n]             | Returns a reference to the element at position `n` in `v`.   |
| `v1` = `v2`        | Replaces the elements in `v1` with **a copy** of the elements in `v2`. |
| `v` = {a, b, c...} | Replaces the elements in `v` with **a copy** of the elements in the comma-separated list. |
| `v1` == `v2`       | `v1` and `v2` are equal if they have same `size()` and each element in `v1` is equal to the corresponding element in `v2`. |

在此需要对`v.push_back(t)`和`v.pop_back()`函数进行一些说明：

```c
	vector<int> v;
	int i = 42;
	v.push_back(i);
	v[0] = 1;
	cout<<v[0]<<endl;  // 1
	cout<<i<<endl;  // 42
	v.pop_back();
	cout<<v.size()<<endl;  // 0
```

如之前所说，`vector`中的elements实际上是存放在堆上的一块连续内存，通过`vector`中的3个迭代器来指向该段内存来实现增删等操作。简单来说，`v.push_back(t)`会经历以下几个步骤（不考虑空间不足时的扩容操作）：

- 给`vector`添加一个`element`，因为此时内存足够，所以在堆上的那块连续内存中不需要执行什么操作。
- 调用object `t`的`copy constructor`将`t`拷贝到`vector`新添加的`element`所指向的地址。注意此时是**深拷贝**，即添加到`element`所指向地址的objec并不是原来的`t`。

这也是为什么`push_bacn(t)`的作用是`Adds an element with value t to end of v.`因为添加进堆上的object并不是`t`，只是`t`的拷贝，拥有`t`的值而已。这也是为什么代码示例中`i`输出还是42的原因。

同样，`pop_back()`并不需要传入参数，因为其作用是移除`v`中的最后一个元素（即`--finish`迭代器减少1），并且调用该元素中的`object`的析构函数进行析构。但是如果该`objec`没有合适的析构函数，例如基本类型`int`，`bool`等，则不会进行其他操作。**但是即使调用了析构函数也只是清除了这段内存上的内容（地址仍保留），但是并没有用`deallocate`回收这段内存，因此仍然可以强制通过`subscript`来拿到这段内存，[here](https://blog.csdn.net/tangle001/article/details/47026989)， [here](https://blog.csdn.net/fao9001/article/details/76255206)。

## `vector size_type`

类似`string`，`vector`也有自己的`size_type`。也就是说，`v.size()`返回的结果是`vector<int>::size_type`类型（可以理解为`unsigned`）。但是需要注意的是，如果想要explicitly使用 `vector<int>::size_type`，那么需要指明这个`vector`中的element的类型：

```c
vector<int>::size_type size; // ok
vector::size_type size;  // error
```

## `Range for`

```c
vector<int> v{1,2,3,4};
for(auto &i : v)  // using reference to modify the elements
  i *= i;
for(auto i : v)  // just printf the value of each element
  cout<<i<<endl;
```

## `subscrit` operations

和`string`一样，也可以通过下标操作来获取`vector`中相应位置的element。不过需要注意的是在C中一般对于数组会利用`for`循环直接给数组赋值，但是在`vector`中不能这样，而是需要通过`push_back()`来添加`element`，在copy值到这个`element`上。例如：

```c
int a[10];
for(int i=0; i<10; i++)  // ok, assign value to each element in an array
  a[i] = i;

vector<int> v;
for(int i=0; i!=10; i++)  //  error, becasue v is an empty vector
  v[i] = i;
// v.push_back(i);  //  need to use push_back(i) to add element;
```

另外，`vector`的下标操作也需要注意`index`必须是一个可以取到的值，不然就是`buffer overflow`问题，这也是leetcode中最常见的崩溃错误。