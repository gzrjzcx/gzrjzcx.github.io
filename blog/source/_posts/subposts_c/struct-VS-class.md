---
title: struct VS. class
date: 2019-02-23 01:03:10
tags: c
categories: C
hide: true
---
# C++: `struct` Vs. `class`
Firstly, in `C++`, **Both `class` and `struct` declare a class**. The `struct` is intended for backward compatibility with `C`(different with `C`, because it is a class).

## Accessibility
- `struct:` have default **`public`** members and bases. It means when inheriting, `struct` defaults to `public` inheritance.
- `class:` have default **`private`** members and bases. It means when inheriting, `class` defaults to `private` inheritance.
  
Both directives allows a mixture of `public`, `protected` and `private` data and member functions.  
In fact, two formats below are absolutely equivalent in every way except their name:
```c
struct Foo
{
	int x;
};

class Bar
{
	public:
		int x;
};
```
## When to use:
- If we want to use structs as plain-old-data(POD) structures without any class-like features, then we need to use `struct`. **Note**, the `POD` means that is a class (whether defined with the keyword struct or the keyword class) without `constructors`, `destructors` and `virtual members functions`.
- Then other situations is suitable to use `class`.
  
## Using `struct` as `lambda` function
```c
struct Compare {bool operator() {...}};
std::sort(collection.begin(), collection.end(), Compare());
```
However, the `lambda` function has been supported in `C++11` by all the major compilers.

## Constructor
Since both directives are `class`, both are able to have `constructor` and `destructor`.  
But some people would like to stick with the `struct` keyword for classes without member functions, because the resulting definition "looks like" a simple structure from C.

# References
- [when to use](https://stackoverflow.com/questions/54585/when-should-you-use-a-class-vs-a-struct-in-c/54596#54596)
- [POD](https://stackoverflow.com/questions/146452/what-are-pod-types-in-c)
- [Constructor](https://stackoverflow.com/questions/1127396/struct-constructor-in-c)
- [overflow](https://stackoverflow.com/questions/32719880/why-is-there-not-an-stdis-struct-type-trait/34108140#34108140)