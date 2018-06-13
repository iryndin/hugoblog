+++
# Description = ""
date = "2018-06-11T19:16:46Z"
title = "Leetcode 53. Maximum subarray"
# menu = "main"
# Categories = ["Development","GoLang"]
Tags = ["leetcode", "dynamic programming", "programming problems"]
+++

From a given array find the maximum contiguous subarray with the largest sum, 
[Leetcode 53](https://leetcode.com/problems/maximum-subarray/description/). 

<!--more-->

## Description

Given an integer array nums, find the contiguous subarray (containing at least one number) which has the largest sum and return its sum.

### Example

```
Input: [-2,1,-3,4,-1,2,1,-5,4],
Output: 6
Explanation: [4,-1,2,1] has the largest sum = 6.
```

### Follow up

If you have figured out the O(n) solution, try coding another solution using the divide and conquer approach, which is more subtle.

## Brute force solution

For each index combination let's calculate sum and then find the largest one. 
Space complexity is `O(1)` and time complexity is `O(n^2)`.

```java
public class Solution {
  public int maxSubArray(int[] nums) {
    if (nums == null || nums.length == 0) {
      return 0;
    } else if (nums.length == 1) {
      return nums[0];
    }

    int maxSum = Integer.MIN_VALUE;
    for (int i=0; i<nums.length; i++) {
      int sum = 0;
      for (int j=i; j<nums.length; j++) {
        sum += table[i][j];
        if (sum > maxSum) {
          maxSum = sum;
        }
      }
    }
    return maxSum;
  }
}
```

## A better way

Now let's think about better approach. Let's take into account following points: 

* we are looking for largest sum, so any negative value will deteriorate final result
* Let's go from the start of the array and sum up all elements. If at some point we have negative sum, then
we can set total sum to current array element. Why? Because any new value will be better than negative sum. Negative element will only deteriorate final sum, so it will be better, zero element is greater than any negative value, the same for positive element. 
* If sum is not negative, then we simply add up next array element. 

So, code will be following:

```java
public class Solution {
  public int maxSubArray(int[] nums) {
    int max = Integer.MIN_VALUE, sum = 0;
    for (int i = 0; i < nums.length; i++) {
      if (sum < 0) {
        sum = nums[i];
      } else {
        sum += nums[i];
      }
      if (sum > max) {
        max = sum;
      }
    }
    return max;
  }
}
```