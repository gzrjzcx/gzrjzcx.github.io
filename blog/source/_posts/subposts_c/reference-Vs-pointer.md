---
title: reference Vs. pointer
date: 2019-02-22 23:44:05
tags: c
hide: true
---
# pointer Vs. reference
Firstly, `reference` is the concept of `C++`, it is a simple reference data type that is less powerful but safer than the pointer type.
```c
int a=2;
int& ra=a; // define a reference ra
std::cout << ra; // 2
```
`Reference` is a sort of **alias** for the target. For example, `ra` is a sort of alias for the `r` in the above example. When we refer to the `ra`, practically refer to the `a`. **`ra` and `a` are conceptually one**. Changing `ra` means changing `a`, and changing `a` means changing `ra`.

> Essentially, `reference` is same as the `pointer`, `reference` is the **grammar sugar** of the `pointer`.

Because from the assembly code, the implementation of `reference` is almost same as pointer. Therefore, **reference is also need to be allocated the memory**.
- `Reference` is the grammar sugar of the `pointer`, it is intended as the `const pointer` which can do the address-of operation(`&`) and the dereference operation(`*`) by the compiler. 
- `Reference` must be **initialized** when it is created, `pointer` not.
- `Reference` is a sort of **alias** of the target, therefore, `reference` must be **non-null**, `pointer` not. 
- `Reference` is the **target value** execute the increment(`++`) or decrement(`--`) operation, but the `pointer` is itself executes the increment or decrement operation.
  

Particularly, the `reference` can be used as the `formal parameter` to avoid the check of if the passed parameter is `0`. For example:
```c
void fun1(int *point)
{
	// need to check whether the point is NULL
     if(!point)
     {
        return;
     }
}

void fun2(int &reference)
{
     // reference is must be non-null 
}
```



**`sizeof(*pointer)` will get the memory space of a pointer, `sizeof(&reference)` will get the referenced object's memory space.**

# References

- [zhihu](https://www.zhihu.com/question/37608201)
- [wiki](https://en.wikipedia.org/wiki/Reference_(C%2B%2B))
- [overflow](https://stackoverflow.com/questions/4995899/difference-between-pointer-and-reference-in-c)
- [assembly perspective](https://www.marvinle.cn/2018/12/10/pointer-reference/)
