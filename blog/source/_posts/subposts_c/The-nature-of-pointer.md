---
title: The nature of pointer
date: 2019-02-22 13:37:59
tags: c
hide: true
---
# What is a pointer
Essentially, what must be emphasized is that **`pointer` is a data type** in `C` context, like `int` data type. Therefore:
- `pointer` need memory space to store.
- `pointer` has its own address of the memory space.

> In computer science, a pointer is a programming language object that stores the memory address of another value located in computer memory. A pointer references a location in memory, and obtaining the value stored at that location is known as `dereferencing` the pointer.  ---[WIKI](https://en.wikipedia.org/wiki/Pointer_(computer_programming))

Given these points, we can say that `pointer` is a data type which is intended to store the memory address. More importantly, we can obtain the value of the `pointer` by `dereference` operation.

# Memory address of pointer
Since we have known that `pointer` is used to store the memory address, then what is the `memory address`? It can be diagrammed by the figure below:
  
![pointer1](/res/c/pointer/pointer1.png)

This is the memory figure of the below code:
```c
unsigned char a = 7; // unsigned char only contain 1 bit
unsigned char* pa = &a; //assign the address of a to pointer b
unsigned char** ppa = &b; //assign the address of the pointer pa to pointer ppa
printf("dereferencing ppa = %d\n", **ppa);
```
It can be explained that the variable `unsigned char a` is initialized with `7`, and stored at `0x00` memory space. Then referencing variable `a` to pointer `pa`. It means that acquiring the address of variable `a` and assigning it to pointer variable `pa`.  
**Note:** here you can see:
- `The value` of pointer `pa` is `0x00`, i.e. the address of variable `a`.
- `The address` of pointer `pa` is `0x01`, i.e. the address of pointer `pa` itself.

It is what we mentioned before, the pointer `pa` is a data type and it has its own address. The `deference operation` means that obtaining the value of the address stored in the `pointer`(i.e., the value of `pointer`).
  
The situation is similar to the pointer `ppa`, it means the pointer `ppa` to the other pointer `pa`. The value of the pointer `ppa` is the address of the pointer `pa`. And the pointer `ppa` has its own address `0x02`.

# The type of the pointer
Firstly, the pointer is intended to store the address, however, the address is only a series of numbers. Therefore, the different types are not necessary for pointers in terms of the pointer itself. However, there is the most important reason that we need different type of pointers in terms of `dereference operation`:  
**`Tell the compiler how to interpret the variable.`**  
  
![pointer2](/res/c/pointer/pointer2.png)
  
Specifically, when we acquire the address from the pointer `pb`(i.e. `0x00`), we only know the variable `b` start from the `0x00` in the memory space. We don't know how many blocks the variable `b` contains. We must tell the compiler that `pb` is a `int*` type pointer, then the compiler will know that we need to obtain the first `4` blocks of this memory to interpret the variable `b`(because the `int` type contains `4` bytes).

## void pointer
Specially, `void pointer` can transfer to arbitrary type pointer(e.g. `int*`, `char*`) with necessary explicit type conversion. But arbitrary type pointer transfer to `void pointer` without any conversion.
```c
void* p;
int* pa = (int *)p; // explicit type conversion
float f = 1.22f;
float* p1 = &f;
p = p1; // without any conversion
```
The reason that why we need to execute the type conversion explicitly is that we need to tell the compiler how many blocks of the memory are used to interpret the variable.
  
Typically, the `void pointer` is intended as the parameters of the function which the type of the pointer is uncertain. For example:
```c
void* memcpy(void *addr1, void *addr2, size_t n);
```
It means arbitrary pointer type can pass to this function, because it is only need to operate the memory, the type of the pointer is inessential. 

# Pointer operation
Firstly, the operation of pointer means the operation of the **`value` of pointer**.
As the figure:
  
![pointer3](/res/c/pointer/pointer2.png)
  
Please note that in this stage, the pointer `pb` is stored in `0x20` instead of `0x04`. Then 
```
pb+1 ==> 0x00 + 4 ==> 0x04
```
We can see that the addition of pointer is the value of the pointer add the `sizeof(type)` bytes. Therefore, it is pointless of the addition of pointer, because it is hard to explain what is in memory `0x04`(or even `0x04` memory block does not exit).  
In some cases, the subtraction is allowed in `C` because we can the offset of two pointers.

# array name Vs. pointer
## Conclusion
Array is array, pointer is pointer, they are different data type. Essentially, pointer is a variable, `array name` is a symbol(constant). 

## Analysis
```c
int main(void) {
    int a[5] = {1,2,3,4,5};
    int (*pa)[5] = &a;
    int* paEle = a;
    //exp1
    printf("%d\n", sizeof(a)); //20
    printf("%d\n", sizeof(paEle)); //8
    //exp2
    printf("%p\n", &a); // 0x7ffdb136eb30 
    printf("%p\n", a);  // 0x7ffdb136eb30 
    //exp3
    printf("%p\n", a);  // 0x7ffdb136eb30 
    printf("%p\n", pa);  // 0x7ffdb136eb30 
    printf("%p\n", a+1);  // 0x7ffdb136eb34
    printf("%p\n", paEle+1);  // 0x7ffdb136eb34
    printf("%p\n", pa+1);  // 0x7ffdb136eb44
    printf("%d\n", *(a+1));  // 2
    printf("%d\n", *(pa+1)); // -1321800892
    return 0;
}
```
- In terms of the `exp1`, From the `sizeof` result, it can be concluded that the `array name` and `pointer` are different. Because if `a` is a pointer, then `sizeof(a)` should be equal to `8`(64-bit machine).  
- As for the reason that the results of `exp2` are equal, it is that when the `array` are passed as the parameters of function, then it will decay to the `pointer`. 
- With respect to the `exp3`, `pa+1`(i.e. the pointer to the array `a`) means the offset will be the **whole array**. This is why the result of `*(pa+1)` is a random value. However, the offset `a+1` and `paEle+1` are one element(4 bytes for `int`).

## Differences
### `1. Declaration:` 
Except the situation that passed as the parameter, `array name` and `pointer` are totally different. Below is a wrong example:
```c
char a[]; //define an array in `a.c` file,
extern char* a; //declare a as a pointer in other file, cannot compile(gcc not sure)
```
### `2. pointer is a variable, array name is a constant`
- The essential of `array name` is the address of the first element of an array. In assembly code, it is a **constant**. Therefore, it can not `++` or `--`. In fact, typically, the `a[i]` will be transfered to `*(a+i)`. 
- `a`, i.e. the address of the first element of array can be acquired **in the compilation stage**. In the assembly code, the address of the array are solid. For example, assuming that the address of `a` is `0x8048000`, then `a[1]` will be transfered to `*(0x8048000 + 4)`, (`int` are 4 bytes). However, pointer is a variable, the value cannot be acquired when compiling.
- Reference to array(`a[1]`) will access the memory `1` time, reference to pointer `*(p+1)` will access the memory 2 times. Because `a` is the constant, what need to do is use constant `a` plus `1` and deference the address to the value. However, pointer is a variable, need to deference it firstly and plus `1`, and then deference the obtained address.

# pointer to array Vs. array of pointers
```c
int *array1[10];  // An array of pointers. Each element of this array is one pointer. 
                  // There are 10 pointers in total
int (*array2)[10];  // A pointer to an array. Each element of this array is one integer.
                    // There are 10 integers in total, and a pointer to this array.
int *(array3[10]);  // same as the first one. An array of pointers.
```
  
> The reason is that the operator precedence of `[](array subscripting)` is higher than `* (dereference)`.

In fact, 
- In the first case, initially is the `array1[10]` operation, i.e. an array `array1[10]`, then is the `*array1[10]` operation, i.e. each element of this array is one int pointer. 
- In the second case, the bracket `()` improves the operator precedence of `*` to the same level of `[]`. The practical associativity order is (\*array2) firstly, i.e., a pointer `array2`, then is an array of ten integers.

## Initialize array of pointers
```c
double *pos[10];
pos[0] = calloc(10*5,sizeof(double));
for(i=1;i<10;i++){
    pos[i] = pos[0] + i * 5;
}
```
Here, we cannot do this: `pos = calloc(10*5, sizeof(double))` directly, because here `pos` is the **`array name`**, it is a **constant**!

## 2-d array of pointers
```c
int (*a[10])[5]; // a is an array of pointers, each pointer to an array of 5 integers.
```
Here, using the bracket `()` to associate the `*` with the first dimension `a[10]`, i.e., the first dimension is an array pointers, and then the second dimension is the `[5]`. In a word, what each element within `a` points to is an array with 5 integers.  
In other words, `a` is an array of pointers, there are 10 pointer within `a` in total, each element is a pointer to an array with 5 integers.

# References:
- [pointer](https://www.zhihu.com/question/24466000)
- [pointer type](https://www.zhihu.com/question/31022750)
- [void pointer](https://blog.csdn.net/geekcome/article/details/6249151)
- [void pointer 2](https://blog.csdn.net/qq_29924041/article/details/54882135)
- [array name and pointer](https://www.zhihu.com/question/23059655)
- [array name assembly](http://blog.chinaunix.net/uid-27004869-id-3301282.html)
- [array of pointers](https://stackoverflow.com/questions/859634/c-pointer-to-array-array-of-pointers-disambiguation)
- [operators in C](https://en.wikipedia.org/wiki/Operators_in_C_and_C%2B%2B#Operator_precedence)