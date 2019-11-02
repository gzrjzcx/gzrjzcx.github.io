---
title: Singly-Linked List
date: 2019-10-03 12:27:40
tags: c
categories: DataStructure
hide: true
---
# Singly-Linked List
> In computer science, a linked list is a linear collection of data elements, called nodes, each pointing to the next node by means of a pointer. It is a *data structure* consisting of a group of nodes which together represent a sequence.  --WIKI

### Define
```c
struct ListNode
{
	int val;
	struct ListNode *next;
};
typedef struct ListNode node;

// or simply define
typedef struct ListNode
{
	int val;
	struct ListNode *next;
}node;  // `node` is the new struct name
```

### Head Node
Typically, there are two ways to initial a linked list: has a head node or hasn't a head node. It can be diagrammed by the below figure:

![linkedlist](/res/c/linkedList/linkedlist.png)
  
`An important note here is that the *head node* is **not** necessary for the linked list, it is leveraged to simply iterate the linked list. But the *root pointer* is essential for the linked list.`

### Usage
Here assuming that we have initialized a linked list without head node, and then we need add the elements to the end in order. For example, we want to store 6,7,8 to the list in order:
```c
node *root=NULL; // a root pointer for the linked list
node *prev=NULL; // a temp node for adding node to the end of the list

int a[3] = {6,7,8};
for(int i=0; i<3; i++)
{
	node *cur = (node *)malloc(sizeof(node));
	cur->val = a[i];
	cur->next = NULL;

	if(!root)
		root = cur; // Need to assign the first node to the root pointer
	else
		prev->next = cur;
	prev = cur;
}
```
The idea is showed by the below figure:
  
![addnode](/res/c/linkedList/addnode.png)

### Reverse List
There are different ways to reverse a linked list, here we mainly introduce the iterative approach.
Assuming we have a linked list from the above example:
```c
node reverse(node *root)
{
	// use three extra pointers for iteration.
	node *prev=NULL, *cur=root, *next=root->next;
	while(next)
	{
		cur->next = prev;
		prev = cur;
		cur = next;
		next = next->next;
	}
	if(!next)
		cur->next = prev;
}
```
The process can be illustrated by the diagram:



Finally, the `cur` node will be the last node, and the `next` will be NULL. Note that in the last step(i.e. next is NULL), we should make the `cur` node points to the `prev` node.

### Remove Nth Node From End
Here, we will introduce a method to remove the nth node from end with only *one pass*: **Using two pointers**.

`The core idea of using two pointers is that when the faster pointer reached the end of the list, the slower pointer will reach the nth node. The gap between two pointers is equal to n.`

```c
node *removeNthFromEnd(node *head, int n){
    node *p1 = head, *p2 = head;
    int count = 1;
    while(p1->next)
    {
        if(count > n)
            p2 = p2->next;
        p1 = p1->next;
        count++;
    }
    if(count == n)
        head = head->next;
    else
        p2->next = p2->next->next;
    return head;
}
```


  

















