---
title: 'Pseudo Random Number: Linear Congruential Generator'
date: 2021-03-28 15:18:12
categories: GameAI
tags: GameAI
hide: true
---

# 伪随机数： 线性同余法的应用

## 1. 背景

在传统游戏AI业务中，经常会用到随机数来避免机械化的行为。比如，以行为树（BehaviourDesigner）实现的AI需要使用到随机数来产生随机行为，从而避免重复动作，增加AI行为的多样性。同时，又因为帧同步的要求，需要保证在不同设备或者回放（重新跑帧逻辑）时产生的结果是一致的。所以，我们通常会选择伪随机数发生器来完成随机要求。

另外，在我们项目中，对于产生的伪随机数还有一些额外要求：

1. **在同一帧内，不同AI个体产生的随机数结果要相互独立**（i.e. 也就是A个体的随机结果和B个体的随机结果间不能存在某种关系）。这样可以避免多个AI走同一套AI逻辑时，如果所处环境很类似，会产生一样的行为的情况。视觉看上去这是多个AI出现了行为一致，很容易被看出来是AI。

   

2. **在同一帧内，同一AI个体多次调用随机数接口产生的随机数结果应该保持不变。**这是因为我们的AI模块中会有难度设置，不同难度下需要通过一个随机概率来调节AI做出某种行为的概率。例如，低难度下AI做出进攻行为的条件应该是`A的攻击力(100)*调节概率(0.8) > threshold`。此时的调节概率是随机出来的值，但是该节点可能会在一帧内执行多次，或者一帧内会有多个节点会使用到这个调节概率。所以，为了保持一帧内的结果一致，需要保证一帧内同一AI个体多次调用随机数接口时产生的结果一致。

   

3. **在不同帧内，同一AI个体产生的结果应该独立。这是最基本最常见的随机数的使用要求。**一般来说，只要合理选择线性同余发生器的参数，则可以在其周期范围内认为其产生的结果是均匀的。

## 2. 线性同余法

线性同余法应该是使用最多的伪随机数方法，如果能够选择合适的参数，对于一般的随机要求是足够的。其核心原理则是通过不断取模在一个模数范围内取得每次的值。只要模数足够大，且是满周期，则可以认为在这个模数范围内取得任一值的概率是相等的。其核心公式为：

$$X_n = (a*X_n-1 + c) % m$$.

这里我们从[gcc]( https://man7.org/linux/man-pages/man3/rand.3.html )的实现上来看：

```c
// 此套参数为 a = 1103515245, c = 12345, m = 32768
static unsigned long next = 1;

/* RAND_MAX assumed to be 32767 */
int myrand(void) {
    next = next * 1103515245 + 12345; // 修改种子
    return((unsigned)(next/65536) % 32768); // 返回伪随机结果
}

void mysrand(unsigned int seed) {
    next = seed;
}
```

这里有几个值得注意的地方：

- 在`myrand()`函数中，`next`表示的就是种子。这里每次调用该函数，会先修改此次随机行为的种子为`next * a + c`。但是按照线性同余法原本的定义，下一次随机的种子应该为`(next * a + ) % m`，但是在gcc的实现中利用了整数的溢出规则，让其超过整形限制时自动实现取模操作。这种方法在滚哈希的时候常用，理论上会比手动取摸更快（寄存器自然操作）。

- 在计算随机结果的时候，先对种子进行了`next/65536`的操作，这一步为了抛弃低位数字。因为高位数字比低位数字具有更长的周期，所以这样会产生更均匀的随机数。

  >  As shown above, LCGs do not always use all of the bits in the values they produce. For example, the [Java](https://en.wikipedia.org/wiki/Java_(programming_language)) implementation operates with 48-bit values at each iteration but returns only their 32 most significant bits. This is because the higher-order bits have longer periods than the lower-order bits (see below). LCGs that use this truncation technique produce statistically better values than those that do not. This is especially noticeable in scripts that use the mod operation to reduce range; modifying the random number mod 2 will lead to alternating 0 and 1 without truncation. 
  >
  > ​																																	---- [wiki]( https://en.wikipedia.org/wiki/Linear_congruential_generator )

简单总结，每次调用`myrand()`函数，都会经历一次修改种子，计算随机结果的操作。只要这样不断更新种子，才能保证取到的伪随机数是真正在周期范围内（m内）只均匀出现过一次的数。很显然，传统的LCGs是不能满足我们对于随机数的要求1和要求2，（要求3天然满足）。因此，我们需要对LCGs进行修改。

## 3. 在每帧开始固定一个随机数种子

我们最开始的想法，是在每帧开始前先固定一个随机数种子，然后对于不同的角色，根据其id修改种子以取得不同的结果。代码如下：

```c
static unsigned long seed = 1;

// 一帧内不同英雄都可能多次调用此函数
int myrand()
{
    if(curFrame != GetFrame())
    {
        seed = seed * 1103515245 + 12345;
        curFrame = GetFrame();
    }
    rand(playerId);
}

int rand(void, playerId) {
    // 往下在取一次种子，同时使用playerId区别不同角色随机的结果
    unsigned long nextSeed = (seed + playerId) * 1103515245 + 12345; 
    return((unsigned)(nextSeed/65536) % 32768); // 返回伪随机结果
}
```

从代码中可以看到，此时会在每一帧开始时先固定一个种子，然后对于不同的`playerId`，在该种子的基础上往后取一次种子，同时以该id作为偏移量。咋一看这样确实满足了背景上的三个条件，但是其实存在问题。在LCGs中，**如果种子固定，那么在该种子上作偏移，始终是存在一定的线性关系**。举个例子，如果此时`myrand(playerId = 1)`的结果为28，那么`myrand(playerId = 2)`的结果大概率为大于28的随机数（说大概率是因为可能存在溢出的情况）。具体可以看下图：

![rand_with_offset](/res/gameAI/randomNumber/rand_with_offset.png)

从图中可以看出，`obj_id = 4`和`obj_id = 11`明显有着同样的“距离”。也就是说，此时不同的玩家，随机到的结果却是始终出现了类似的结果。如果体现在AI上，就是不同的AI个体，但是在随机做一些动作时，总是会随到同样的动作。或者随机往某个方向移动时，总是会随机到类似的方向。这就导致AI的拟人化很差，人类玩家一样就能看出来AI。