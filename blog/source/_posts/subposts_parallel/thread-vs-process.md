---
title: 'thread Vs. process'
date: 2019-02-21 16:29:01
tags: Parallel
hide: true
---

# Difference between `thread` and `process`
---
1. Process is the instance of the program, thread is the subset of process.
2. One program has at least one process, one process has at least one thread.
3. Process runs in a separate memory space, thread runs in a shared memory space.
4. Process has its own address space, once crash, have no impact for other processes. Thread only has its own stack but shared other resources, once crashed, whole program crashed.

# What is thread
---
{% blockquote %}
	*Thread is **subset** of the process.*
{% endblockquote %}

#### Shared Memory System
In terms of the thread, we need to know the base of **shared memory**:  
In the shared memory system, there is **only one single address space** across the whole memory system.
{% blockquote %}
	- Every CPU/core can read/write all memory locations in the system.
	- Only one logical memory space.
	- All CPUs/cores refer to a memory location using the same address.
{% endblockquote %}
#### Multiple threads
Given these points, typically, we can say that the term of `multiple threads` is mainly used in the **shared memory system**. Because threads can access the shared data within the whole memory system by `read/write` method instead of message. It is the communication in different threads.  
{% blockquote %}
	Each thread has its own `stack` memory, but `heap` memory segment are shared.
{% endblockquote %}
#### Data
There are two different kinds of data in `multiple threads` contexts:
- `Shared data:` can be accessed by all threads.
- `Private data:` can be accessed by the owning thread.

Each thread has its own copy the executable(i.e., the codes), however, different threads can follow different control flow through the program because each thread has its own program counter(i.e. **register**).
  
Typically, *run one thread per CPU/core*. The register contains the address of the instruction being executed at the current time. Therefore, a program can control different threads.

#### Synchronization
By default, threads execute **asynchronously**.  
Each thread executes through the program instructions **independently** of other threads. Therefore, it is very important to guarantee the correct order of shared variables processed.

# What is process
---
{% blockquote %}
	*A process is the instance of a computer program that is being executed.*
{% endblockquote %}
It contains the program code and its activity. It has distinct address space, different from other executing instance of program in what it has separate resources.

#### Distributed-Memory Architecture
Distributed-memory refers to a multiprocessor computer system in which each processor has its own **private memory**. Computational tasks can only operate on local data, and if remote data is required, the computational task must communicate with one or more remote processors.  
Not only a single memory space used like `shared memory`. Which can avoid `**data race conditions**`.

#### Data
All variables are **private**. Each process has access only to its own data.  
All processes run the same program(their own copy), each process has a separate copy of data.  
Processes can follow different control paths through the program, depending on their process ID

#### Communication & Synchronization
By sending and receiving message to other processes to finish communication. Therefore, there is no `data race`, rather than `dead lock`:
- A `synchronous send` is not completed until the message has started to be received.
- An `asynchronous send` completes as soon as the message has gone.
- `Receives` are usually synchronous - the receiving process must wait until the message arrives




















