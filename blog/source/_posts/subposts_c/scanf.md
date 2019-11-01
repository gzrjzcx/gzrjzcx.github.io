---
title: scanf and its left white-space character problem
date: 2019-10-04 14:13:44
tags: c
hide: true
---
# Several tricks of scanf

The `scanf` function is used to scan input from the stdin stream according to the format specification. In some cases, the collocation of other input functions(e.g., `gets`) will cause some problems. Assuming we will read below contents from the stdin to the program:

```
3
cap
to
cat
```
  
A common method is taking advantage of the `scanf` function to read the integer firstly; thereafter, read the string for each line. The codes are shown below:

```c
int main()
{
	int n;
	scanf("%d", &n);
	printf("%d\n", n);
	while(n--)
	{
		char str[100];
		gets(str);
		printf("%s\n", str);
	}
	/* results:
	3

	cap
	to
	*/
}
```
  
Obviously, there is an unexpected `\n` after the first `scanf`. The reason is the `scanf("%d")` will stop reading when it encountered the WSC(white-space characters, i.e. **white-space, tab, newline**). The first input(i.e. 3) in the input file can be presented as `3\n`. Therefore, the `scanf` has only read the `3`, but the `\n` has been left in the stream buffer. However, the `gets()` function read the string from the buffer until encountering the `\n`. Therefore, the first `gets` read the `\n` which left in the first `scanf`.
  
### Consume the left '\n'
In conclusion, to get the correct input, we need to clean the left `\n` in the buffer. There are two methods to do this:

```c
int main()
{
	int n;
	scanf("%d", &n);
	getchar();  // use getchar() to consume the left '\n'
	/* 
	char ch;
	scanf("%c", ch); // or use scanf to consume the left '\n'
					 // scanf("%c") or scanf("%d") can recognize the WSC
	*/
	printf("%d\n", n);
	while(n--)
	{
		char str[100];
		gets(str);
		printf("%s\n", str);
	}
}
```
  
> `fflush(stdin)` cannot clean the buffer in Linux system because it is not a standard function and supported by all compiler.

### Ignore the left '\n'
Alternatively, we can force the `scanf` to ignore the left '\n':

```c
int main()
{
	int n;
	scanf("%d", &n);
	printf("%d\n", n);
	while(n--)
	{
		char str[100];
		scanf("%s", str);
		printf("%s\n", str);
	}
}
```
  
The core idea is that the `scanf("%s")` will match a sequence of non-white-space characters. In other words, it can skip the left `\n`.

```c
int main()
{
	int n;
	scanf("%d\n", &n); // equal to scanf("%d ", &n);
	printf("%d\n", n);
	while(n--)
	{
		char str[100];
		gets(str);
		printf("%s\n", str);
	}
}
```
  
An alternative is adding the WSC to the format string of `scanf`.
> Note: the meaning of the WSC in the format string of `scanf` is to explicitly read and ignore as many whitespace characters as it can[[1]](https://stackoverflow.com/questions/19499060/what-is-the-effect-of-trailing-white-space-in-a-scanf-format-string), instead of expecting a `\n` new line sign. Therefore, the `scanf("%d\n")` means after reading an integer, it will continue to read characters, discarding all whitespace until it sees a non-whitespace character on the input. That non-whitespace character will be left as the next character to be read by an input function.
  
Therefore, the first `scanf("%d\n")` will read the integer `3` and discard the `\n`, then encounter the next non-white-space character(i.e. `c`) and stop. But the `c` will be left to the buffer to the next input function(e.g. `gets()`).
  
**Note:** `gets()` is unsafe because it is potential to cause overflow for the string. Therefore, we can try to use `fgets(str,MAXLEN,stdin);`

### References and relative reading:
- [scanf detail](https://blog.csdn.net/xuelongyinyue/article/details/47358193)
- [scanf("%c") vs scanf("%s")](https://blog.csdn.net/Wmll1234567/article/details/82463573)
- [man scanf](http://www.man7.org/linux/man-pages/man3/scanf.3.html)