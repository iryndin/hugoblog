+++
date = "2018-06-11T19:37:06Z"
title = "Leetcode 198. House robber"
Tags = ["leetcode", "dynamic programming", "programming problems"]
+++

Another easy task is about smart [house robbery approach](https://leetcode.com/problems/house-robber/description/).

<!--more-->

## Description

You are a professional robber planning to rob houses along a street. Each house has a certain amount of money stashed, the only constraint stopping you from robbing each of them is that adjacent houses have security system connected and **it will automatically contact the police if two adjacent houses were broken into on the same night**.

Given a list of non-negative integers representing the amount of money of each house, determine the maximum amount of money you can rob tonight **without alerting the police**.

### Example 1
```
Input: [1,2,3,1]
Output: 4
Explanation: Rob house 1 (money = 1) and then rob house 3 (money = 3).
             Total amount you can rob = 1 + 3 = 4.
```

### Example 2
```
Input: [2,7,9,3,1]
Output: 12
Explanation: Rob house 1 (money = 2), rob house 3 (money = 9) and rob house 5 (money = 1).
             Total amount you can rob = 2 + 9 + 1 = 12.
```

## Dumb solution

Initially, I came up with a pretty straitforward (and dumb) solution for the task. 
Just stupid checks of all variants of robbery, here is the code: 

```java
class Solution {
  public int rob(int[] nums) {
    if (nums == null || nums.length == 0) {
      return 0;
    } else if (nums.length == 1) {
      return nums[0];
    } else if (nums.length == 2) {
      return Math.max(nums[0], nums[1]);
    } else if (nums.length == 3) {
      return Math.max(nums[0]+nums[2], nums[1]);
    }
    int[] a = new int[nums.length];
    a[0] = nums[0];
    a[1] = nums[1];
    a[2] = nums[0] + nums[2];
    int maxRob = Math.max(Math.max(a[0],a[1]), a[2]);
    for (int i=3; i<nums.length; i++) {
      a[i] = Math.max(nums[i]+a[i-2], nums[i]+a[i-3]);
      if (a[i] > maxRob) {
        maxRob = a[i];
      }
    }
    return maxRob;
  }
}
```

## Better way 

But more smart solution can be as easy as following (and it has `O(1)` space complexity, 
as well as `O(n)` time complexity):

```java
class Solution {
  public int rob(int[] nums) {
    int a = 0, b = 0;
    for (int num : nums){
      int tmp = Math.max(num+a, b);
      a = b;
      b = tmp;
    }
    return Math.max(a, b);
  }
}
```

Let's explain what goes on here. Variable `a` stores prev previous value, `b` contains previous value. And `b` is updated only if new value is bigger than its previous value. This is the key to this solution. 