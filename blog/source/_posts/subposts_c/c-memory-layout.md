---
title: Static keyword & C memory layout 
date: 2019-02-22 21:54:33
tags: c
categories: C
hide: true
---
# Static keyword
The `static` keyword in C language, is mainly intended to control the scopes of the variable.
- `static global variable`: It is a **file-level** operation. The static global variable will cannot be found by other `.o` files when linking them.  Only the file where the variable are defined can use these global variable.
- `general global variable`: It is can be found by any file by using the `extern` directive.
- `static local variable`: It is stored in the `.BSS` segment so that the lifecycle will be the whole runtime of the program. And then it will be initialized to `0`.
- `general local variable`: It is stored in the `slack` segment, then it will be destroyed in the end of the current scope(e.g., the current function).
  
> **Note**: The `static` and `extern` directive are both the **file-level** operations, which means that they are used to prevent/support the different files to access the described variables.
  
Particularly, it also makes the **local variable** stored from `stack` area to the `static` area(i.e., `.BSS` segment) in the `C` memory layout below; therefore, the lifecycle of `static local variable` will last to the **entire running period of the program**. Additionally, this `static local variable` will be initialized to `0` because they are stored in the `.BSS` segment.

# C memory layout
```
High Addresses ---> .----------------------.
                    |      Environment     |
                    |----------------------|
                    |                      |   Functions and variable are declared
                    |         STACK        |   on the stack.
base pointer ->     | - - - - - - - - - - -|
                    |           |          |
                    |           v          |
                    :                      :
                    .                      .   The stack grows down into unused space
                    .         Empty        .   while the heap grows up. 
                    .                      .
                    .                      .   (other memory maps do occur here, such 
                    .                      .    as dynamic libraries, and different memory
                    :                      :    allocate)
                    |           ^          |
                    |           |          |
 brk point ->       | - - - - - - - - - - -|   Dynamic memory is declared on the heap
                    |          HEAP        |
                    |                      |
                    |----------------------|
                    |          BSS         |   Uninitialized data (BSS) 静态区
                    |----------------------|   
                    |          Data        |   Initialized data (DS) 文字常量区
                    |----------------------|
                    |          Text        |   Binary code 代码区
Low Addresses ----> '----------------------'  
```
## Stack segment
The `stack` segment contains the `local variables` and the `parameters` from functions mainly. Its memory are allocated and released  by the **`compiler`** automatically. It is fast but the available memory is limited. 
  
When a function is called, a `stack frame` will be created in the `stack` segment:
- Each function has one stack frame.
- Stack frames contain the function's local variables, parameters and return value.
  
The operation of `stack` segment is similar to the `stack` in `data structure` context. 
Therefore, the memory space of the `stack` segment is **contiguous**.

Function variables are pushed onto the stack when called, and the functions variables are popped off the stack when return. The `stack pointer register` tracks the top pf the stack.

> **The lifecycle of each stack frame is the scoop of function**. Once the function has been executed, the memory of this stack frame will be released.

## Heap segment
It is used to allocate the memory at runtime. It allocated and released by the memory management functions like `malloc` and `free` **manually**. Its total memory space depends on the system. It is slow but the available space is adequate. The `heap` segment is shared by all shared libraries and dynamically loaded modules in a process. 

**Note:** It is totally different to the `heap` in the `data structure` context. In fact, its operation model is similar to the `linked list`. Therefore, the memory space of the `heap` segment is **not contiguous**.

> **Its maximum lifecycle is the whole runtime of the program**, but must be released manually.

## BSS(static segment)
Also known as the `Uninitialized data segment` or **`static`** segment.
- All **uninitialized** `global variable` and `static variable` stored here.
- All variables in `static` segment initialized by the `0`(pointer with the `null pointer`)
  
**Note:** the `static` means the variable in the segment would not be changed with the function calling, instead of the meaning that the value of these variables would not be changed.
  
> **Its lifecycle is the whole runtime of the program**, and will be released by the system automatically.

## DS(.data segment)
Also known as the `Initialized data segment`:
- The **`explicitly initialized global and static` variables are stored here**.
- The size of this segment is determined by the size of the values in the program's source code and does not change at runtime.
- It has read-write permission so the value of the variable of this segment can be changed at run time.
- This segment can be further classified into an initialized read-only area and initialized read-write area.

> **Its lifecycle is the whole runtime of the program**, and will be released by the system automatically.

## Text segment
This area contains binary of the compiled program.
- The text segment is a read-only segment that prevents a program from being accidentally modified.
- It is sharable so that only a single copy needs to be in memory for frequently executed programs such as text editors etc.

> **Its lifecycle is the whole runtime of the program**, and will be released by the system automatically.

# References:
- [Memory Layout of C program](https://aticleworld.com/memory-layout-of-c-program/)
- [Static key word in C](http://www.cnblogs.com/dc10101/archive/2007/08/22/865556.html)
- [Verify memory layout of C program](https://wongxingjun.github.io/2015/07/25/C%E7%A8%8B%E5%BA%8F%E7%9A%84%E5%86%85%E5%AD%98%E5%B8%83%E5%B1%80/)