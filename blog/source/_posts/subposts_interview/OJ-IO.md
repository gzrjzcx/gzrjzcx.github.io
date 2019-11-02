---
title: OJ IO
date: 2019-04-05 22:34:12
tags: Interview
categories: Interview
hide: true
---

# OJ IO
The motivation of this blog is learning the method of implementation of input and output of OJ mode. Because I used to practice programming in `LeetCode`, it is very verdant for me to program in the `NowCoder`. In the light of this, several common IO situations of `OJ` mode will be discussed here, as well as an practical example.

In conclusion, **input data will be written in line by line from `stdin` in terms of `OJ` mode.** Particularly, the different types of data will be taken into account with respect to `C` language. 

## Int
### 1. Multiple sets of input data without a specific limitation of quantity. Write data until the end of input file.
```c
int main()
{
	int m;
	while( scanf("%d", &m) != EOF )
		...
}
```
Here, `EOF` is a defined macro with value of `-1`. The return value will be the total number of characters written successfully. Otherwise, it will be a negative value. Therefore, the `while` loop is intended to write data from `stdin` continuously(line by line) until the end of the file.

In addition, due to the value of `EOF`(i.e. `-1`), the code can be simplified as below:
```c
int main()
{
	int m;
	while( ~scanf("%d", &m) )
		...
}
```
Because of `~-1=0`, above code is the other form of aforementioned write method.

### 2. Multiple sets of input data without a specific limitation of quantity. Write data until the end of input file. But a special symbol is used as the ending sign.
```c
int main()
{
	int m;
	while( scanf("%d", &m) != EOF && m != 0) // == while( ~scanf("%d", &m) && m != 0)
		...
}

//input data example:
// 3
// 10
// 81
// 0  // the end of sign, we should not deal with this line.
```

### 3. Given a specific N multiple sets of input data, then write data these N sets of data.
```c
int main()
{
	int N;
	int a,b;
	scanf("%d", &N);
	while(N--)
	{
		scanf("%d %d", &a, &b);
		...
	}
}
```

This is the most common situation in fact, the practical example is on the basis of this situation.

## String 
### 1. No Whitespace(` `), Newline(`\n`), Carriage Return(`\r`), and Tab(`\t`) at each String.

```c
// input: 
// abc def

char s1[100], s2[100];
scanf("%s %s", s1, s2);
```

### 2. No Newline(`\n`) or Carriage Return(`\r`) at each String.
```c
// input: 
// Hello World!

char s1[100];
gets(s1);
```
`gets()` function will be used to write the `String` in terms of this situation. Because we are facing a training environment, the length of input will be guaranteed. Therefore, the risk of `gets` function can be ignored. We can also use `fgets()` to assign a specific length of `String` will be write. Additionally, the return value of `NULL` means the end of file or error generated.

> Note that using `scanf` before `gets` may cause an unexpected problem, click [here](/_post/subposts_c/scanf) to check the detail.

## An example
In this case, we will read 3 sets of data, each set contains one line(2 integers) to describe the value of `rows` and `cols`. Then are values of a matrix with `rows` row and `cols` columns.

```c
// input:
// 3              // There are 3 datasets in total
// 5 10           // There are 5 rows and 10 columns in the first dataset
// AAAAAADROW
// WORDBBBBBB
// OCCCWCCCCC
// RFFFFOFFFF
// DHHHHHRHHH
// WORD           // The target word of the first dataset
// 3 3            // There are 3 rows and 3 columns in the second dataset
// AAA
// AAA
// AAA
// AA             // The target word of the second dataset
// 5 8            // There are 5 rows and 8 columns in the third dataset
// WORDSWOR
// ORDSWORD
// RDSWORDS
// DSWORDSW
// SWORDSWO
// SWORD          // The target word of the third dataset

void input(int *rows, int *cols, char *target, char **matrix){
	scanf("%d %d\n", rows, cols); // MUST ADD '\n'!!
	for(int i=0; i<*rows; i++)
	{
		matrix[i] = (char *)malloc(*cols * sizeof(char));
		gets(matrix[i]);
	}
	gets(target);
}

int main(int argc, char* argv[]){
	int N;
	scanf("%d\n", &N); // MUST ADD '\n'!!
	int count = 0;
	int rows, cols;
	char *target = (char *)malloc(9 * sizeof(char));
	char **matrix = (char **)malloc(99 * sizeof(char *));
	for(int i=0; i<99; i++)
		matrix[i] = (char *)malloc( 99 * sizeof(char));
	while(N--)
	{
		input(&rows, &cols, target, matrix);
	}
	return 0;
}
```
An important note here is that if we read integer by `scanf` firstly, and then read String by `gets`, we must add the `\n` in the `scanf` function. Because the `scanf("%d\n", &N);` will stop writing once finishing the integer written. Therefore, the last new-line character will not be consumed by `scanf`. 

# References:
- [OJ IO](https://blog.csdn.net/qq_27848507/article/details/53145100)
- [~scanf](https://www.quora.com/What-does-this-statement-mean-while-scanf-d-d-d-a-b-n)
- [Newline and Return](https://www.cnblogs.com/yunf/archive/2011/04/20/2021830.html)
- [scanf not deal with new-line character](https://stackoverflow.com/questions/14484431/scanf-getting-skipped) 