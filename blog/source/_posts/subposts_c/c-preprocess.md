---
title: C preprocess
date: 2019-02-22 23:23:13
tags: c
categories: C
hide: true
---
# Preprocessor
The C preprocessor is a macro preprocessor which allows you to define macros that transforms program **before it compiled**. These transformations can be inclusion of header file, macro expansions etc. There are three common preprocessor strategies:
- Macros using `#define`
- Including Header Files
- Conditional Compilation

## 1. Macros using
A macro is a fragment of the code that is given a name. We can use that fragment of code in our program by using this name. 

> The core idea of the `#define` macro is the **text substitution**. It's better that adding bracket for each variable.

A simple macro which is used to get the maximum value from two values:
```c
#define min(a,b) ((a)<(b)?(a):(b))
```

For example:
```c
#define SQR(X) X*X
...
a /= SQR(k+m)/SQR(k+m);
```
The result will be: `a = a / k+m*k+m/k+m*k+m`. Obviously, this is not what we want. Here, the problem is that the macro is the pure **`text substitution`**. It means that `X` will be substituted by `k+m` in any case. Therefore, the better method of using macro is adding bracket for each variable:
```c
#define SQR(X) ((X)*(X))
...
a /= SQR(k+m)/SQR(k+m);
```
Then the final result is `((k+m)*(k+m))/((k+m)*(k+m))`.
  
A more complicated example of macro is using macro to [handle error](https://github.com/MarioMartReq/PS_GroupWorkC/blob/master/include/macros.h):
```c
#define CHECK_ERROR(error_status, error_func)\
({\
	if(error_status!=ERROR_SUCCESS)\
	{\
		if(error_status!=ERROR_SUCCESS_HELP)\
		{\
			fprintf(stderr, "ERROR::%s:%d:%s: Function %s returned error %d!\n",\
						__FILE__, __LINE__, __func__, error_func, error_status);\
		}\
		return(error_status);\
	}\
})
```

### Macro Vs. Inline
- Inline is just a request to the compiler in `C++`, macro is the preprocessor in `C`.
- Inline replaces a call to a function **with the body** of the function, macro is expanded by the preprocessor before compilation(macro is **text substitution**).
- Inline can do the type checked, macro can not.
- Inline is used to optimize the program, macro is used to achieve `inline` function before `C99` standard.

## 2. Including head files
```c
/* "" will find the .h file in the current project firstly, then to the system*/
#include <stdio.h>. // used for standard .h
#include "my_header.h" // used for user-defined .h
```
The `#include` preprocessor directive will replace the above line with **the contents of `stdio.h`** head file which contains the function and macro definitions.

## 3. Conditional Compilation
Conditional compilation is the process of defining compiler directives that cause different parts of the code to be compiled, and others to be ignored. This technique can be used in a cross-platform development scenario to specify parts of the code that are compiled specific to a particular platform.
```c
#ifdef MACRO     
     conditional codes
#endif
```
Here, the conditional codes are included in the program only if `MACRO` is defined.
  
**Note:** the `MACRO` is not necessary defined in the program, it also can be defined out of the program by compilation. For example, if we use `icc` compiler, we can use `-D` flag to define the custom macro to the program. [Example](https://github.com/gzrjzcx/openmp_affinity)
  
```c
#if expression
   conditional codes if expression is non-zero
#elif expression1
    conditional codes if expression is non-zero
#elif expression2
    conditional codes if expression is non-zero
...
#else
   conditional if all expressions are 0
#endif
```
Here, expression is a expression of integer type (can be integers, characters, arithmetic expression, macros and so on). The conditional codes are included in the program only if the expression is evaluated to a non-zero value.
  
```c
#if defined BUFFER_SIZE && BUFFER_SIZE >= 2048
  conditional codes
```
The special operator `#defined` is used to test whether certain macro is defined or not. It's often used with `#if` directive.

# References
- [Preprocessor](https://www.programiz.com/c-programming/c-preprocessor-macros)







