---
title: 'Turn on/off lamps'
date: 2019-02-21 00:26:09
tags: Interview
hide: true
---

# 1. Turn on/off lamps
---
Imaging that there are N lamps(`1,2,3...`), and in the first time there are **lit**:
- The 1st person will turn off all of the N lamps;
- The 2nd person will turn on the 2nd lamps and the multiple of 2 lamps;
- The 3rd person will turn off the 3rd lamps and the multiple of 3 lamps;
...
- the nth person will turn on/off(**reverse operation**) the nth lamps and the multiple of nth lamps.

**Question:** Print the ID of the lamps which are lit in order after the nth person did the reverse operation. E.g., After the 8th operation, the lamps with ID `2 3 5 6 7 8` are lit.

### Solution
The core idea of this question is that we only need to take **the operation times of lamps** into account. For example, if we know the 5th lamps are turned on/off 4 times, then it is still lit. Therefore, the most thing is to know the times of turn on/off operation for each light.
  
Then note that the `i` person can only turn on/off the `i` and the multiple of `i` lamps. In other words, as for the lamps `i`, only the person of the submultiple of `i` can turn on/off the lamps. For example, with respect to the lamp `8`, only the `1 2 4 8` person can turn on/off this lamp. Therefore, **the number of the submultiple of lamp `i` is the times of the operations of lamp `i`.**
  
Given these points, the question can be transfered to find the submultiple for the `i` lamp.  

### References
- [csdn](https://ask.csdn.net/questions/203246)