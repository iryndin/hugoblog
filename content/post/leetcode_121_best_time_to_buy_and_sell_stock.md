+++
# Categories = ["Development","GoLang"]
date = "2018-06-09T12:19:43Z"
title = "Leetcode 121. Best time to buy and sell stock"
Tags = ["leetcode", "dynamic programming", "programming problems"]
# Description = ""
# menu = "main"

+++

Let's condsider easy dynamic problem: best time to buy and sell stock. 
This is [Leetcode 121. Best Time to Buy and Sell Stock](https://leetcode.com/problems/best-time-to-buy-and-sell-stock/description/).

<!--more-->

## Description

Say you have an array for which the `i-th` element is the price of a given stock on day i.

If you were only permitted to complete at most one transaction (i.e., buy one and sell one share of the stock), design an algorithm to find the maximum profit.

Note that you cannot sell a stock before you buy one.

### Example 1
```
Input: [7,1,5,3,6,4]
Output: 5
Explanation: Buy on day 2 (price = 1) and sell on day 5 (price = 6), profit = 6-1 = 5.
             Not 7-1 = 6, as selling price needs to be larger than buying price.
```

### Example 2
```
Input: [7,6,4,3,1]
Output: 0
Explanation: In this case, no transaction is done, i.e. max profit = 0.
```

## Brute force solution

Without a lot of thinking we can come up with brute force solution. 
We buy stock on some particular day, and then for the next days we check profit if we sell this stock. Thus we will find the maximum profit. Time complexity is `O(n^2)`, while space complexity is `O(1)`. 

```java
class Solution {
  public int maxProfit(int[] prices) {
    int result = 0;
    for (int i=0;i<prices.length-1;i++) {
      int maxProfit = maxProfit(prices, i);
      if (maxProfit > result) {
        result = maxProfit;
      }
    }
    return result;
  }

  // Calculate max profit is stock is bought on day idx
  // and sold on one of the next days
  static int maxProfit(int[] prices, int idx) {
    int diff = 0;
    for (int j=idx+1; j<prices.length; j++) {
      diff = Math.max(diff, prices[j]-prices[idx]);
    }
    return diff;
  }
}
```

## More smart solution

However, we can come up with more smart solution, that has time complexity only `O(n)`, 
i.e. only one array pass is required.

Here is the idea. We got max profit when we buy on lowest price, and  sell on highest price. 
So let's track minimum price and max profit. Let's traverse through price array, 
and check profit and min price with every new array element. If we found new min price and 
new max profit, then update our variables. Space complexity for this approach is `O(1)`.

```java
class Solution {
  public int maxProfit(int[] prices) {
    if (prices == null || prices.length < 2) {
      return 0;
    }
    int minPrice = Integer.MAX_VALUE-1;
    int maxProfit = 0;
    for (int i=0;i<prices.length;i++) {
      int todayProfit = prices[i] - minPrice;
      if (prices[i] < minPrice) 
        minPrice = prices[i];
      }
      if (todayProfit > maxProfit) {
        maxProfit = todayProfit;
      }
    }
    return maxProfit;
  }
}
```






