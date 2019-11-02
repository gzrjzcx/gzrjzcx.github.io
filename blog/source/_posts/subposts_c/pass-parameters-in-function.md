---
title: Pass parameters in function
date: 2019-02-22 23:57:46
tags: c
categories: C
hide: true
---

If you are not clear about the concept of `pointer`, please click [here](https://www.hellscript.cc/2019/02/22/subposts_c/The-nature-of-pointer/):

# pass by value Vs. pass by reference Vs. pass by pointer
Firstly, in `C` language, there is only **`pass by value`** method to pass parameters. Therefore, we must use `pointer` to access the local variable of other function. `pass by reference` is the strategy supported by `C++`.

## pass by value
```c
void f( int  p){
	printf("\n%x",&p);			//12ff44
	printf("\n%x",p);			//10
	p=0xff;
}
void main()
{
	int a=0x10;
	printf("\n%x",&a);			//12fef4
	printf("\n%x\n",a);			//10
	f(a);
	printf("\n%x\n",a);			//10
}
```
In terms of the `passing by value` strategy, the variable will be copy to the memory as a new local variable(this is why the address of `a` is different than the `p`). Therefore, modifying the value of the `p` in the function `f` cannot modify the value of the `a`.

### pass by reference
```c
void f( int  &p){
	printf("\n%x",&p);			//12ff44
	printf("\n%x",p);			//10
	p=0xff;
}
void main()
{
	int a=0x10;
	printf("\n%x",&a);			//12ff44
	printf("\n%x\n",a);			//10
	f(a);
	printf("\n%x\n",a);			//ff
}
```
`passing by reference` strategy will pass the address of the actual parameter to the formal parameter, therefore, in fact, the variable `p` in the function `c` is same as the variable `a`. If we modify the value of `p`, the value of `a` also could be modified. This is why the address of both `p` and `a` are `12ff44`, and the value of `a` has been modified to `ff` after calling the function `f`.

### pass by pointer
```c
void f( int  *p){
	printf("\n%x",&p);			//12fef4
	printf("\n%x",p);			//12ff44
	printf("\n%x",*p);			//10
	p=0xff;
}
void main()
{
	int a=0x10;
	printf("\n%x",&a);			//12ff44
	printf("\n%x\n",a);			//10
	f(a);
	printf("\n%x\n",a);			//ff
}
```
`pass by pointer` will also pass the address of the variable `a` to the `pointer p` in the function `f`. **Note** that `12fef4` is the actual address of the `pointer p` itself. And then the value of the pointer is `12ff44`, i.e. the address of the `a`. Therefore the **dereference** operation will get the value of the variable `a`. And this method will also can change the value of `a` after calling the function `f`.

`**Note: If you want to modify the value of a pointer, you still need to pass the address of this pointer to the other context(function).**`
  
```c
typedef struct ListNode
{
    char word[100];
    struct ListNode *next;
}node;

int main()
{
	...
	node *prev;
    insert(root, cur, &prev); // must pass the address of the pointer
}

void insert(node *root, node *cur, node **prev)
{
	...
	*prev = cur;
	// must pass the address of the pointer so as to modify the value of the original pointer
}
```
  
Here, if you only pass the `prev` instead of the `&prev`, it will be the `pass by value`. The pointer `prev` will be copied in the `insert()` scoop(stored in the `stack` segment, will be collected in the end of the function). Therefore, the actual value `prev` pointer has not been modified.
### Reference
- [details](https://blog.csdn.net/whzhaochao/article/details/12891329)
- [zhihu](https://www.zhihu.com/question/51582974)