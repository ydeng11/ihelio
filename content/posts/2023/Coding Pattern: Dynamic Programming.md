---
title: "Coding Pattern: Dynamic Programming"
date: 2023-11-30T16:28:21-04:00
draft:
categories: 
    - Programming
tags: 
    - Algorithm
    - LeetCode
---

# Preface

**Facing the Dynamic Programming Challenge**

Like many others, I initially found Dynamic Programming (DP) on LeetCode daunting and perplexing. However, this challenging journey led to profound insights. My initial misconception was that DP was all about complexity, but I learned it's fundamentally about simplifying complex problems into manageable segments. Here's my journey into understanding DP and why it's a crucial tool in a programmer's toolkit.
# Unraveling Dynamic Programming

**The Essence of DP**

At its heart, Dynamic Programming is a method to efficiently solve problems by breaking them into smaller, interdependent subproblems. This approach is akin to Divide-and-Conquer and Recursion, but with a unique twist: the subproblems in DP are not independent. The solution to one affects the others, creating a web of dependencies that must be navigated carefully.

**Two Key Questions of DP:**

1. What is the optimal (maximum or minimum) outcome?
2. How many distinct solutions exist?

DP provides structured frameworks to tackle these questions effectively.

# The DP Framework: A Three-Step Process

**1. Define States and Variables:**

- Identifying 'states' is crucial. In DP, a state represents a specific condition or scenario within the problem. For instance, in calculating Fibonacci(10), the states include Fibonacci(9) and Fibonacci(8).

**2. Transition Between States:**

- The core of DP lies in figuring out how these states evolve from one to another. Using our Fibonacci example, the transition is defined by Fibonacci(10) = Fibonacci(9) + Fibonacci(8). It's about finding a pattern or a rule that governs this evolution.

**3. Establish Base Cases:**

- Base cases act as the starting point for the recursive journey of DP solutions. They are typically straightforward and known. In the Fibonacci sequence, they are Fibonacci(1) = 1 and Fibonacci(2) = 1.

## Approaches: Top-down and Bottom-up

**Top-down (Memoization):**

- This approach feels natural as it mirrors human problem-solving: start from the end goal and work backward. Memoization, an optimization technique, stores results of expensive function calls and returns the cached result when the same inputs occur again.

**Bottom-up (Tabulation):**

- More iterative and often faster, Bottom-up systematically solves and stores the results of all subproblems. It eliminates the need for recursion, thereby saving memory overhead.

### Understanding Time and Space Complexity

Both approaches converge on time complexity but can differ in execution speed and memory usage. The complexity often depends on the number of states and transitions involved.

**Time Complexity:** Generally, it correlates with the total states processed. For instance, in a DP solution with states $dp(i, j)$, if $i \leq m$ and $j \leq n$, the time complexity is typically $O(mn)$.

**Space Complexity:** Varies between the two methods. Top-down's space complexity includes the recursion stack, while Bottom-up's aligns closely with its time complexity due to pre-allocation of state storage.
# Classic Problems

## 1D Dynamic Programming

1D DP refers to the problems we could use one state variable to define the state, such as [House Robber](https://leetcode.com/problems/house-robber/).

```python
# Bottom-up approach
class Solution:
    def rob(self, nums: List[int]) -> int:
        if len(nums) == 1: return nums[0]
        # There are n houses, so the number of states is n
        dp = [0] * len(nums)
        # The state variable is i, and dp[i] states the max money robbed at ith house
        # The base case at 0th and 1th house
        dp[0] = nums[0]
        dp[1] = max(nums[0], nums[1])
        for i in range(2, len(nums)):
	        # The recurrence relation 
	        # 1. Not robbing ith house, the max money should be equal to i-1
	        # 2. Robbing ith house, then (i - 1)th house cannot be robbed
            dp[i] = max(dp[i - 1], dp[i - 2] + nums[i])
        # return dp[n - 1]
        return dp[-1]
```
## Multi-Dimension Dynamic Programming

Multi-Dimension DP represents the problem we need to use more than 1 state variable to represent the state such as [Maximum Score from Performing Multiplication Operations](https://leetcode.com/problems/maximum-score-from-performing-multiplication-operations/)

```python
# Top-down approach
class Solution:
    def maximumScore(self, nums: List[int], multipliers: List[int]) -> int:

		# Using two state variables, i and left, i is index of multipliers and left is the pointer indicating the potential number take from left end
        @cache
        def helper(i, left):
	        # Base case, if we have used up all operations, then we have no value gain any more
            if i == len(multipliers):
                return 0
            # compute the position of right pointer we could use
            right = len(nums) - 1 - (i - left)
            # the transition pattern includes two cases
            # we take the number either from left or right
            return max(helper(i + 1, left + 1) + nums[left] * multipliers[i], helper(i + 1, left) + nums[right] * multipliers[i])
        # we should return helper(0, 0) since it is on the top given our state variable - when no operations are executed yet
        return helper(0, 0)
```

## Path and Count DP

**Count Problems: The Quest for Total Solutions** Count problems in Dynamic Programming focus on determining the total number of possible solutions to a problem. A classic example is the [Coin Change II problem on LeetCode](https://leetcode.com/problems/coin-change-ii/). Here, the challenge is to find out how many distinct combinations of given coins can sum up to a target amount. It's not just about whether a solution exists, but rather how many different ways there are to achieve it. This type of problem leverages DP's ability to explore and count all potential combinations efficiently.

**Path Problems: Charting Routes through Constraints** Path problems, while similar to Count problems, often present a slightly different challenge. These problems typically involve navigating through a grid or matrix to find all possible paths from a start point to an end point. The key to applying DP effectively here lies in the movement constraints. For instance, if movement is limited to only right and down in a grid, DP is an ideal approach. However, if movement includes all four directions (up, down, left, right), the problem complexity increases, potentially requiring alternative strategies like Breadth-First Search (BFS) or Depth-First Search (DFS). In these unrestricted movement scenarios, using DP might lead to revisiting the same cell multiple times, making it less efficient.

![](https://i.imgur.com/BWiozAJ.png)

Let's use [Coin Change II](https://leetcode.com/problems/coin-change-ii/) to learn how to solve Count DP.
 
```python
class Solution:
    def change(self, amount: int, coins: List[int]) -> int:
        # Top Down approach
        # We use the target amount and i as the state variables indicating the number of combinations we can make for this amount using the coins starting from i in the list
        @cache
        def helper(amount, i):
	        # When the amount is 0, which means we have a viable combination and we should return 1
            if amount == 0:
                return 1
            # if the amount is less than 0, then we don't have a viable combination, in all other cases, we should keep looking
            if amount < 0:
                return 0
            
            out = 0
            # We will try each coin starting at i
            for i in range(i, len(coins)):
	            # we accumulate the possible combinations we find as the final output
	            # since we have infinite number of coins, we could try first possible coin till we cannot - this is similiar with DFS. Once we tried all possible combinations starting at ith, we could move to (i + 1)th coin
	            # remember one common pitfall about Count DP is to deduplicate the solution, since we return 1 once we have a viable solution, we need make sure our search path will not duplicate - which means once we tried all possible ways starting with ith coin, we should not include it in our following search
                out += helper(amount - coins[i], i)
            
            return out
        # The problem case should be helper(amount, 0) given how we define our state variables (see the beginning)
        return helper(amount, 0)
```

Let's use [63. Unique Paths II](https://leetcode.com/problems/unique-paths-ii/) to learn Path DP.

```python
class Solution:
	# Use Bottom-up approach
    def uniquePathsWithObstacles(self, obstacleGrid: List[List[int]]) -> int:
        # If the start is blocked, then we cannot move 
        if obstacleGrid[0][0] == 1: return 0
        
        m, n = len(obstacleGrid), len(obstacleGrid[0])
        # Preallocate the space for dp states
        # dp[i][j] represents the uniquePath at (i, j) in the grid
        dp = [[0] * n for _ in range(m)]

		# base case, there is only 1 way at (0, 0)
        dp[0][0] = 1

		# go through all possible states
        for i in range(m):
            for j in range(n):
	            # if there is a blocker, it cannot be reached
                if obstacleGrid[i][j] == 1:
                    continue
                # since the agent can only move down or right, the number of unique paths will be the sum of the unique paths at its top and left
                # it is dp[i][j] = dp[i - 1][j] + dp[i][j - 1], here we need handle the out of boundary case
                if i - 1 >= 0:
                    dp[i][j] += dp[i - 1][j]
                if j - 1 >= 0:
                    dp[i][j] += dp[i][j - 1]
        # the problem case will be dp[m][n] where is the bottom-right cell
        return dp[-1][-1]
```
## Iteration and Inaction Recurrence Relation

In some cases, we need to iterate through some possibility to find the optimal solution, like the Coin Change problem we saw in [[#Path and Count DP]].

Another case is we might need skip the current operation.

Like [188. Best Time to Buy and Sell Stock IV](https://leetcode.com/problems/best-time-to-buy-and-sell-stock-iv/), we need decide if we should buy, sell or hold to maximize our gain in the stock.

```python
class Solution:
	# Use top-down approach
    def maxProfit(self, k: int, prices: List[int]) -> int:

		# there are three state variables
		#  - leftTransactions: how many transactions we can still make
		#  - i: the ith day in our trading period
		#  - holding: whether we have the stock at our hands
		# it represnts the max gain we can get given the above state
        @cache
        def helper(leftTransactions, i, holding):
	        # if we cannot make any transactions or pass the trading window, we cannot make any more profit, so the max gain is 0
            if leftTransactions == 0 or i == len(prices):
                return 0
            
            out = 0
            # One choice is we do nothing on ith day, so we could pass our state to the next day
            out = max(out, helper(leftTransactions, i + 1, holding))
            # if we don't hold the stock, we could choose to buy it on ith day and our gain need subtract the cost of purchase
            if holding == 0:
                out = max(out, -prices[i] + helper(leftTransactions, i + 1, 1))
            # if we have the stock, we could choose to sell it and the profit will be price on ith day
            elif holding == 1:
                out = max(out, prices[i] + helper(leftTransactions - 1, i + 1, 0))
            return out
        # thus we should return helper(k, 0, 0) indicating the max gain we can get when we can make k transactions starting day 0 without holding it
        return helper(k, 0, 0)
```

## Kadane's Algorithm

See [My introduction to Kadane's Algorithm](https://ihelio.today/posts/2023/coding-pattern-kadanes-algo/)