+++
Tags = ["leetcode", "dynamic programming", "programming problems"]
date = "2018-06-09T01:49:55Z"
title = "Leetcode 70. Climbing stairs"
# Categories = ["Development","GoLang"]
# menu = "main"
+++

Let's condsider easy dynamic problem: climbing stairs. 
This is [Leetcode 70. Climbing stairs](https://leetcode.com/problems/climbing-stairs/description/).

<!--more-->

## Description

You are climbing a stair case. It takes n steps to reach to the top.  
Each time you can either climb 1 or 2 steps.  
In how many distinct ways can you climb to the top?  
Note: Given n will be a positive integer.

### Example 1

```
Input: 2  
Output: 2  
Explanation: There are two ways to climb to the top.  
1. 1 step + 1 step  
2. 2 steps  
```

### Example 2

```
Input: 3
Output: 3
Explanation: There are three ways to climb to the top.
1. 1 step + 1 step + 1 step
2. 1 step + 2 steps
3. 2 steps + 1 step
```

## Analysis

Number of distinct ways for each stair is a sum of number of distinct ways for previous stair and 
number of distinct ways for previous-previous stair, recursively it can be expressed as following:  
`ways(n)=ways(n-1) + ways(n-2)`

## Recursive solution

```java
class Solution {
  public int climbStairs(int n) {
    switch(n) {
      case 0: return 0;
      case 1: return 1;
      case 2: return 2;
      default: return climbStairs(n-1) + climbStairs(n-2);
    }
  }
}
```

This recursive solution will do many repeating computations, but this is most clear and concise way
to express solution. Now let's consider more economical approach. 

## Better way

This is better solution, with time complexity `O(n)` and space complexity `O(1)`:

```java
class Solution {
  public int climbStairs(int n) {
    if (n <= 2) return n;
    int prevPrevStair = 1, prevStair = 2;
    int curStair=0;
    for (int i=2; i<n; i++) {
      curStair = prevStair + prevPrevStair;
      prevPrevStair = prevStair;
      prevStair = curStair;
    }
    return curStair;
  }
}
```

## Links

## Links

See also [Leetcode 746. Minimal cost climbing stairs]({{< ref "leetcode_746_min_cost_climbing_stairs.md" >}})


