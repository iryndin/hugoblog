+++
date = "2017-04-17T11:38:06+03:00"
title = "Timus 1005. Stone pile"
tags = ["programming problems", "timus", "brute force"]
+++

This is a first post which describes how to solve some particular programming task. Today we will consider 
[Stone pile](http://acm.timus.ru/problem.aspx?space=1&num=1005) task from [Timus](http://acm.timus.ru/). 
It is clear and easy. The task sounds like: 

# Task 

[Stone pile](http://acm.timus.ru/problem.aspx?space=1&num=1005)

## Task

You have a number of stones with known weights w<sub>1</sub>, …, w<sub>n</sub>. 
Write a program that will rearrange the stones into two piles 
such that weight difference between the piles is minimal.

## Input

Input contains the number of stones n (1 ≤ n ≤ 20) and weights of the stones w<sub>1</sub>, …, w<sub>n</sub> 
(integers, 1 ≤ w<sub>i</sub> ≤ 100000) delimited by white spaces.

## Output
Your program should output a number representing the minimal possible weight difference between stone piles.

## Limits

Time limit: 1.0 second

Memory limit: 64 MB

# Solution

The easiest approach is to check all possible stone pile variants and to test which one has minimal weight difference. 
I.e. this is `brute force` approach.

How can we generate all these variants? So, we have 2 piles - first and second. 
Let's designate that stone is in the first pile with value `0`, and that stone is in the second pile with value `1`. 
Let's imagine that we have 3 stones, so let's generate all possible stone placements in 2 piles: 

* 0 0 0 (all 3 stones (w<sub>1</sub>, w<sub>2</sub>, w<sub>3</sub>)are in the first pile)
* 0 0 1 (w<sub>1</sub>, w<sub>2</sub> are in 1st pile, w<sub>3</sub> is in the 2nd pile)
* 0 1 0 (w<sub>1</sub>, w<sub>3</sub> are in 1st pile, w<sub>2</sub> is in the 2nd pile)
* 0 1 1 (etc...)
* 1 0 0 
* 1 0 1
* 1 1 0
* 1 1 1 (all 3 stones (w<sub>1</sub>, w<sub>2</sub>, w<sub>3</sub>)are in the 2nd pile)

Also we can see that regarding the weight difference first 4 variants are identical to the next 4 variants of stone 
placements. This means that is it sufficient to check only half of all possible stone placements, i.e. `n/2`. 

Also we can see that stone placements are bit sequences. So all possible placements will be 2<sup>n</sup>. 
But since we can check only half of them, then we need to check only 2<sup>n-1</sup> placements. 
In Java/C++ it will be equal to `1 << n-1`.

So, summarizing all that, we will have following algorithm:

* Read input values
* Generate maximum number of variants we want to check: `LIMIT = 1 << n-1`
* Loop from `1` to `LIMIT`, incrementing by `1` on each loop cycle. Loop variable designates a particular placement variant.
* In each loop generate weight sum for each stone pile regarding bit order in the loop variable and check if this placement generates minimal weight difference.

# Solution code

```java
import java.io.*;
import java.util.*;

public class Timus1005
{
   public static void main(String[] args)
   {
      Scanner scanner = new Scanner(System.in);
      
      int n = scanner.nextInt();
      int[] stones = new int[n];
      
      for (int i=0; i<n; i++) {
          stones[i] = scanner.nextInt();
      }
      
      int diff = bruteForce(stones);
      
      PrintWriter out = new PrintWriter(System.out);
      out.println(diff);
      out.flush();
   }
   
   static int bruteForce(int[] stones) {
       final int N = stones.length;
       final int LIMIT = 1 << N-1;
       int diff = Integer.MAX_VALUE;
       
       for (int i=0; i<=LIMIT; i++) {
           int sum1=0, sum2=0;
           int k=i;
           for (int j=0; j<N; j++) {
               if ((k & 0x1) == 0) sum1+=stones[j];
               else sum2+=stones[j];
               k = k>>1;
           }
           int diffi = (sum1>=sum2) ? sum1-sum2 : sum2-sum1;
           if (diffi<diff) diff = diffi;
       }
       return diff;
   }
}
```

