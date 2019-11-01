---
title: Typical error of pointer
date: 2019-02-23 00:15:35
tags: c
hide: true
---
# Dangling references Vs. Memory leak
## Dangling references
> A dangling reference(i.e., `wild pointer` or `dangling pinter`) is a reference to an object that no longer exists or cannot access.

For example:
```c
int* p = new int;
delete p;

int i = *p; // error, p has been deleted!
```
Practically, if there are multiple pointer to the same memory, once one of the pointer has released the memory and other pointers still reference it, this is `dangling references`.

### Uninitialized pointer
In some cases, the uninitialized pointer is intended as the `wild pointer`:
```c
char* str;  // wild pointer
static char* str; // not wild pointer because it is initialized to 0 in the .bss memory segment
```
**Note:** In this case the pointer `str` to a random memory(i.e. `garbage value`) instead of `NULL`. Therefore, `wild pointer` and `NULL pointer` are different.
- `NULL pointer` to address `0`.
- `wild pointer` to a random address(garbage).


### A typical example
```c
int main()
{
	int* i = func();
}

int* func(void)
{
    int num = 1234;
    /* ... */
    return &num; //return an address of a local variable
}
```
Here, we try to return the address of the local variable `&num`. However, `num` is a local variable which stored at `stack` segment, once the function have been executed, this memory will be released. In this stage, `i` is the `dangling reference`.  
If we want to return the pointer to `num`, a feasible method is to guarantee the lifecycle of `num` is the whole runtime. E.g., we can set the `num` as `static int num`.

## Memory leak
> `Memory leak` refers to the `heap` memory segment. It means that the allocated memory by `malloc` dynamically cannot be released by `free` correctly. Then this memory block can not be reused until the end of the program.

# Conclusion
In a word, below is a safer method to avoid the `dangling pointer` and `wild pointer`:
```c
int* p = NULL; //initialize to address 0 or pointer to valid memory
p = (int*)malloc(sizeof(int));
...
free(p);
p = NULL; // reference to NULL to avoid the later dereference operation of p
```
**Note:** here the `free` function only means free the memory **instead of delete `p`**. Pointer `p` is a data type, which will be destroyed until out of the scope.

# References
- [wild pointer](https://www.cnblogs.com/submarinex/archive/2013/03/02/2940169.html)
- [wild pointer 2](https://www.quora.com/What-is-the-difference-between-a-wild-and-a-dangling-pointer-in-C)
- [shared_ptr](https://www.jianshu.com/p/e4919f1c3a28)
- [dangling pointer](https://stackoverflow.com/questions/5900165/what-is-the-difference-between-garbage-and-dangling-references)
- [memory leak](https://blog.csdn.net/yuxikuo_1/article/details/41326899)
