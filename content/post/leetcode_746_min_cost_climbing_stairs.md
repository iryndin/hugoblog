+++
Tags = ["leetcode", "dynamic programming", "programming problems"]
# Description = ""
date = "2018-06-09T02:16:40Z"
title = "Leetcode 746. Min cost climbing stairs"
# menu = "main"
# Categories = ["Development","GoLang"]
+++

Let's condsider another easy dynamic problem: minimal cost of climbing stairs. 
This is [Leetcode 746. Min Cost Climbing Stairs](https://leetcode.com/problems/min-cost-climbing-stairs/description/).

<!--more-->

## Description

On a staircase, the `i-th` step has some non-negative cost `cost[i]` assigned (`0` indexed).

Once you pay the cost, you can either climb one or two steps. You need to find minimum cost to reach the top of the floor, and you can either start from the step with index `0`, or the step with index `1`.

### Example 1
```
Input: cost = [10, 15, 20]
Output: 15
Explanation: Cheapest is start on cost[1], pay that cost and go to the top.
```

### Example 2
```
Input: cost = [1, 100, 1, 1, 1, 100, 1, 1, 100, 1]
Output: 6
Explanation: Cheapest is start on cost[0], and only step on 1s, skipping cost[3].
```

### Note

* `cost` will have a length in the range `[2, 1000]`.
* every `cost[i]` will be an integer in the range `[0, 999]`.

## Analysis

Once we can climb one or two steps, for `n-th` stair the minimal cost will be minimal of two values: 
cost for `n-1` step and cost for `n-2` stair. Or, recursively, this can be expressed as:  
`minCost(n) = min(minCost(n-1),minCost(n-2))`.

## Recursive solution

```java
class Solution {
  public int minCostClimbingStairs(int[] cost) {
    if (cost == null || cost.length == 0) return 0;
    else if (cost.length == 1) return cost[0];
    int LEN = cost.length;
    return Math.min(minCost(cost, LEN-1), minCost(cost, LEN-2));
  }

  // Calculate minimal cost to reach stair with index i (i is 0-based index)
  static int minCost(int[] cost, int i) {
    switch (i) {
      case 0: return cost[0];
      case 1: return cost[1];
      default: return cost[i] + Math.min(minCost(cost, i-1), minCost(cost, i-2));
    }
  }
}
```

What is good for this solution, as with almost all recursive solutions, that it is concise and expressive. But for pretty long array it will take tons of time to complete. For Leetcode, this solution gets `TLE` - `Time Limit Exception`. So, let's consider another solution. 

## Better way

Here is solution with `O(n)` time complexity and `O(1)` space complexity:

```java
class Solution {
  public int minCostClimbingStairs(int[] cost) {
    if (cost == null || cost.length == 0) return 0;
    else if (cost.length == 1) return cost[0];
    int LEN = cost.length;
    int prevPrevStairMinCost = cost[0], prevStairMinCost = cost[1];
    int minCost = 0;
    for (int i=2; i<LEN; i++) {
      minCost = cost[i] + Math.min(prevStairMinCost, prevPrevStairMinCost);
      prevPrevStairMinCost = prevStairMinCost;
      prevStairMinCost = minCost;
    }
    return Math.min(prevStairMinCost, prevPrevStairMinCost);
  }
}
```

## Links

See also [Leetcode 70. Climbing stairs]({{< ref "leetcode_70_climbing_stairs.md" >}})
