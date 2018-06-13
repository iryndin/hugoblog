+++
date = "2018-06-13T17:19:11Z"
title = "Leetcode 303. Immutable range sum query"
Tags = ["leetcode", "dynamic programming", "programming problems"]
+++

Calculate sum of array elements for any given indexes, keeping source array immutable at the same time 
[Range Sum Query - Immutable](https://leetcode.com/problems/range-sum-query-immutable/description/).

<!--more-->

## Description

Given an integer array nums, find the sum of the elements between indices i and j (i ≤ j), inclusive.

### Example

```
Given nums = [-2, 0, 3, -5, 2, -1]

sumRange(0, 2) -> 1
sumRange(2, 5) -> -1
sumRange(0, 5) -> -3
```

### Note

* You may assume that the array does not change.
* There are many calls to sumRange function.

## Caching solution

Cache sum of array elements for each combination of indexes into `HashMap`. 
Space complexity is `O(n^2)`, time complexity is `O(n^2)`.

```java
class NumArray {

  Map<String, Integer> cache = new HashMap<>();

  public NumArray(int[] nums) {
    for (int i = 0; i < nums.length; i++) {
      int sum = 0;
      for (int j = i; j < nums.length; j++) {
        sum += nums[j];
        cache.put(i+","+j, sum);
      }
    }
  }

  public int sumRange(int i, int j) {
    return cache.get(i+","+j);
  }
}
```

## We can do better

Sum of array elements between indexes `i` and `j` can be represented as following:  
`sum(i,j)=sum(0,j) - sum(0,i-1)`. We see that it is represented via zero index based sums. 
So we can calculate these sums once, and then use it for our calculations. 

```java
class NumArray {

  int[] zeroBasedSum;

  public NumArray(int[] nums) {
  	zeroBasedSum = new int[nums.length];
  	int sum = 0;
    for (int i = 0; i < nums.length; i++) {
      sum += nums[i];
      zeroBasedSum[i] = sum;
    }
  }

  public int sumRange(int i, int j) {
    return zeroBasedSum[j] - (i-1 < 0 ? 0 : zeroBasedSum[i-1]);
  }
}
```

Obviously, space complexity is `O(n)`, as well as time complexity. 