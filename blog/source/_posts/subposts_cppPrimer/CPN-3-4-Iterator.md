---
title: 'C++Primer 3.4: Iterator'
date: 2019-11-29 12:40:33
categories: cppPrimer
tags: cpp
---

# Iterator

迭代器（iterator）实际上是一种对`遍历容器`操作的抽象和封装。因为STL中提供了各种各样的容器，很多算法都需要`遍历容器中的所有元素`来实现。但是这些容器的底层实现各不相同，例如`vector`依靠数组就可以实现，因此`vector`的迭代器是一个`Random access iterator`；但是`list`的迭代器则是一个`Bidirectional（双向） iterator`，其并不像`vector`那样可以直接通过`subscript`操作取得任意位置的element（因为数组是连续内存，但是链表并不是存储在一块连续内存）。由此可见，**不同类型的容器对于`遍历容器（access any elements in a container）`这个操作的implementation是不同的。**因此，可以看到，迭代器实际上就是对这一操作过程进行了封装，使得不同的容器在实现各种算法时不需要考虑其具体的遍历操作如何实现。换句话说，**STL中通过迭代器将容器和算法解耦，将容器和算法有机地结合起来。**实际上，这也是一种设计模式——`迭代器模式`，即将遍历序列的操作和序列底层的实现相分离。

迭代器（iterator）的具体实现其实没有规定，不过STL的迭代器一般情况下都通过对泛型指针进行封装来实现。在STL中通过对一些操作符例如`->`，`*`，`++`进行重载，从而实现了类似指针的操作方式。但是迭代器本身是一个`class template`，而C中的指针则是一个`data type`。迭代器根据其可执行的操作（即重载不同的操作符），可以分为5种[类型](http://www.cplusplus.com/reference/iterator/)。其中`Random access iterator`类型和C中的指针（pointer）操作基本相同。因此，可以说**C语言中的指针实际上是一个 `Random access iterator`。**

> **Conclusions:**
>
> - `iterator`是`class template`，但是`pointer`是`data type`。

## Using Iterators

对于`Random access iterator`，其常用操作可以总结如下：

| Operations       | Explanation                                                  |
| ---------------- | ------------------------------------------------------------ |
| `v`.begin()      | Returns an iterator **pointing to the `first element`** in the sequence. |
| `v`.end()        | Returns an iterator **pointing to the `off-the-end`** element in the sequence. |
| `*`iter          | Returns a `reference` to the element denoted by the iterator `iter`. |
| iter`->`mem      | Deferences `iter` and fetches the member named `mem` from the undelying element. Equivalent to `(*iter).mem`. |
| `++`iter         | Increments `iter` to refer to the next element in the container. |
| `--`iter         | Decrements `iter` to refer to the previous element in the container. |
| iter1 `==` iter2 | Two iterators are equal if they denote the same element or if they are the `off-the-end` iterator for the same container. |
| iter1 `< `iter2  | One iterator is less than another if it refers to an element whose `position in the container is ahead of the one` referred to by the other iterator. **The iterators must refer to elements in the same container or one `off-the-end` of the container.** |
| iter `+/-` n     | Adding(substracting) an integral value `n` to(from) an iterator yields an `iterator` that many elements forward(backward) within the contains. The resulting iterator must **denote elements in the same container or off-the-end element**. |
| iter `+=/-=` n   | Assigns to `iter` the value of adding `n` to, or substrcating `n` from, `iter`. |
| iter1 `-` iter2  | Returns the `distance` between two iterators. The `distance` means the `position difference` between two elements w.r.t. two iterators. Two iterators must **denote elements in the same container or off-the-end element**. The reuslt is a `signed` integel type. |

首先要说的是，`begin()`和`end()`两个方法都是会返回一个`iterator`，例如：

```c
vector<int> v{0,1,2,3,4};
auto b = v.begin();  // b points to the first element
vector<int>::iterator e = v.end();  // must use `vector<int>`
// e points to the `off-the-end`
```

使用`auto`关键字要求compiler自动推断`b`的类型，`b`则会被推断为`vector<int>::iterator`迭代器类型，并且`b`指向了第一个element，即`0`。但是`end()`则会返回一个指向`off-the-end` element的迭代器。注意这个`off-the-end` element是一个**并不存在**的element，它只是被用来作为已经处理完了所有elements的一个标记。也就是说，**`off-the-end` element是最后一个element的下一个element。** 如果一个container为空，那么`begin()`和`end()`返回的迭代器都指向`off-the-end` element，这样的迭代器也被叫做`off-the-end iterators`。

因为STL中的iterator是通过对泛型指针进行封装来实现，所以其重载了`*`操作符来实现`dereference`操作，即通过`*`可以取得迭代器所指向的underlying object。同样地，`->`也被重载为获取`deference`和`member access`的结合。另外，需要注意的是对于迭代器，其`++`和`--`操作都是被重载为**`指向下一个element或上一个element`**，即将迭代器的位置向前后向后移动一位，也就是说类似于指针的加法。例如：

```c
vector<string> sv{"hello", "world"};
for(auto it = sv.begin(); it!=sv.end(); ++it)
{
  if(!it->empty())  //  equivalent to (*it).empty()
    cout<<*it<<endl;
}
```

上例中`++it`指的是让迭代器`it`指向下一个element，即从`hello`变为`world`。另外，`it->empty()`则完全等同于`*(it).empty()`。因为`*it`得到的结果是一个`string`，在access这个`string`的内部成员`empty()`。

> **Tips:**
>
> 对于STL容器，尽量使用`it.begin() != it.end()`的形式来判断循环结束，而不是`it < it.end()`。因为只有`Random access iterator`重载了`<`，`>`等运算法，其他的容器可能没有重载此操作符。但是所有的库容器都重载了`==`和`!=`操作符。