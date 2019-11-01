---
title: dot operator Vs. arrow operator
date: 2019-02-23 00:11:13
tags: c
hide: true
---
# C structure:  dot operator(.) VS. arrow operator(->)
> Both operators `.` and `->` are used to access the structure members. In fact, the `->` operator is **syntactic sugar** of the `.` operator. For example, `student->name` is same as `(*student).name`.
  
## Difference:
- `Maintainability`: arrow operator `->` is more easier to keep track of which variables are pointers and which are not. It is beneficial to keep memory management.
- `Efficiency`: In some cases the `->` operator is more efficient than `.`.

## Example
For example, assuming that we have a `Student` structure:
```c
/* structure declaration */
typedef struct student {
                        char srollno[10];
                        char sclass[10];
                        char name[25];
                        char fname[25];
                        char mname[25];
                        char add[200];
        }Student;
```
Then we need to access the member of the `Student` structure using **`.`** firstly:
```c
int main(void)
{
    Student a = {"35M2K14", "cs", "Christine", "James", "Hayek",
                  "Post Box 1234, Park Avenue, UK"};
 
    printf("Student a Information:\n");
    dot_access(a);    /* entire 'a' is passed */
    arrow_access(&a); /* just pointer to  Student is passed */
    return 0;
}
 
/* entire Student 'a' is copied into Student 'stu' */
/* 'stu' is a variable of Student, not a pointer */
void dot_access(Student const stu)
{
    /* Let's access members of 'a' using dot operator */
 
    printf("roll no.: %s\n", stu.srollno);			//35M2K14
    printf("class: %s\n", stu.sclass);				//cs
    printf("name: %s\n", stu.name);					//Christine
    printf("father's name: %s\n", stu.fname);		//James
    printf("mother's name: %s\n", stu.mname);		//Hayek
    printf("And address: %s\n", stu.add);			//Post Box 1234, Park Avenue, UK
}

void arrow_access(Student const *stu)    /* 'stu' is a pointer-to-Student */
{
    /* Let's access members of 'a' using arrow operator */
 
    printf("roll no.: %s\n", stu.srollno);			//35M2K14
    printf("class: %s\n", stu.sclass);				//cs
    printf("name: %s\n", stu.name);					//Christine
    printf("father's name: %s\n", stu.fname);		//James
    printf("mother's name: %s\n", stu.mname);		//Hayek
    printf("And address: %s\n", stu.add);			//Post Box 1234, Park Avenue, UK
}
```
We can see that the results of both methods are same. However:
- In `.` mechanism, the lump sum 300 bytes of `Student a` are copied from calling function to the function parameter. 
- In `->` mechanism, just pointer to `Student a`is copied to the pointer-to-Student `*stu` at called function. **Only 8 bytes**. And using this pointer, we accessed the original contents of Student ‘a’ and displayed information.

# References
- [Difference](https://www.sanfoundry.com/c-tutorials-difference-using-dot-arrow-operator-accessing-structure-members/)
- [Overflow](https://stackoverflow.com/questions/10036381/arrow-operator-vs-dot-operator)