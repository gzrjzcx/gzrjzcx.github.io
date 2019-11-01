---
title: 'char* VS char[]'
date: 2019-02-22 22:59:41
tags: c
hide: true
---

# Conclusion
`char*` and `char[]` are different **types** in most cases, but it's not immediately apparent in all cases. Essentially, `char*` means a pointer to a constant stored in the `.data` segment, `char[]` is a `character array` data type.
  
More information about the difference between `array` and `pointer` [here](https://www.hellscript.cc/2019/02/22/subposts_c/The-nature-of-pointer/).

## Implementation
### Method 1: `char*`
```c
char* a = "abc";
```
When executing the above instruction, there are 3 steps in total:
1. Create a pointer `a` to char type and store it in **`stack`** segment in [C memory layout](https://www.hellscript.cc/2019/02/22/subposts_c/c-memory-layout/). 
2. Allocate the necessary memory in **`DS`** segment for the string `abc`.
3. Return the address of this memory as the value and assign to the pointer `a`.
Finally, it is a `char*` pointer to an initialized data stored in `DS` segment. 
  
In fact, if now we set `char* aa = "abc"`, then only will execute the step 1 and 3, because the string constant `abc` has been created in the `DS` segment. 
  
Therefore, it is can be assigned other string like below operation:
```c
a = "def"; // It is legal because a is a pointer
```
However, because the `abc` is constant, stored in the **`DS`** segment(i.e. `data segment`), **it can not be modified**. In other words, the **subscript operation** are illegal in the below example:
```c
char* a = "abc";
printf("%c\n", a[0]); // It is illegal because a is a pointer
```

#### Method 2: `char[]`
```c
char b[3] = "abc";              // it is string, has '\0' in the end as default
char b[3] = {'a','b','c'};      // it is character array, has no '\0' in the end as default
```
`char[]` is the **character array** in C language. When executing the above instruction, there are only 2 steps in total:
1. Declare an array with `char` type, please note in this context `b` is an array, if you are not clear about the difference between `array` and `pointer`, [click here](https://www.hellscript.cc/2019/02/22/subposts_c/The-nature-of-pointer/). This moment, the array variable `b` stored in the `stack` segment, and: **`&b == b == &b[0]`**.
2. Assign each character of `123` to the each element of array `b`.
Finally, the value of this **array** is equal to `abc`, instead of a pointer to point the constant like `char*`. Because it is an `array` type, it is contrary with `char*`: the assignment with string constant is illegal, but the subscript operation is legal.
```c
b = "def"; // reassign with string constant will cause crash
printf("%c\n", b[0]); //legal
```

### Array decay
In terms of a special situation, like using they as the **function parameters**. Specifically, if an expression of type `char[]` is provided where one of type `char*` is expected, the compiler automatically converts the `array` into a `pointer` to its first element.
  
For example, in the function `printSomething` expects a pointer is passed, but if you try to pass an array to it like this:
```c
char s[10] = "hello";
printSomething(s);
```
Then the compiler pretends that your code like this:
```c
char s[10] = "hello";
printSomething(&s[0]);
```

### Dynamic memory
```c
char* s = (char *)malloc(10*sizeof(char));
s = "abc"; // legal
s = "def"; // legal
printf("%c\n", s[0]); // legal
```
This method defines a pointer that point an allocated memory. Therefore, it is legal for `s` to be reassigned by the constant string like `def`. Also, it can do the subscript operation.

### References
- [Array decay](https://stackoverflow.com/questions/10186765/what-is-the-difference-between-char-array-vs-char-pointer-in-c)
- [`char*` Vs. `char[]`](https://blog.csdn.net/ksws0292756/article/details/79432329)














