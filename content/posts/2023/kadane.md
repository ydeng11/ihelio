---
title: "Coding Pattern: Kadane's Algo"
date: 2023-11-30T23:37:39-05:00
draft: 
categories: 
    - Programming
tags: 
    - Algorithm
    - LeetCode
---

**Imagine you're walking along a path that has treasure chests and traps.** Some chests have gold coins, and some traps take away coins. You want to find the part of the path where you can get the most coins.

**The key idea**: It is a dynamic programming algorithm for finding the maximum contiguous sum subarray in a given array. It is a simple and efficient algorithm that works by maintaining two variables:

- `current_sum`: This variable stores the maximum sum of a contiguous subarray ending at the current index.
- `max_sum`: This variable stores the maximum sum of any contiguous subarray in the array.

The algorithm works by iterating over the array and updating the `current_sum` and `max_sum` variables as follows:

```
current_sum = max(array[i], current_sum + array[i])
max_sum = max(current_sum, max_sum)
```

If the current element is greater than the current sum, then the current sum is updated to the current element. Otherwise, the current sum is updated to the current sum plus the current element. The `max_sum` variable is always updated to the maximum of the current sum and the `max_sum` variable.

At the end of the iteration, the `max_sum` variable will contain the maximum sum of any contiguous subarray in the array.

**Illustration using an example**:

Let's say the path (or array) looks like this, where positive numbers are treasure chests (gains) and negative numbers are traps (losses):

```css
[-2, 1, -3, 4, -1, 2, 1, -5, 4]
```

1. Start at the first spot: -2. Your total goes down by 2.
2. Move to the next spot: +1. Now you're at -1.
3. Next is -3. Now you're at -4.
4. Now, you find a big treasure chest of 4! This takes your total back to 0. But since the total before this was negative, it's better to start counting from this chest. Reset the total to 4.
5. Continue on, adding and subtracting as you hit treasures and traps.
6. By the end, you'll find that the best stretch of the path is [4, -1, 2, 1], which gives you 6 coins.

**Graphical Representation**:

```
Path:     -2  |  1  |  -3  |  4  |  -1  |  2  |  1  |  -5  |  4
---------------------------------------------------------------
Total:    -2     -1      -4      4       3       5       6      1      5
Max:      -2      1       1      4       4       5       6      6      6
```

You can see from the table above, we keep track of our running total and the maximum total we've seen so far. The highest number in the 'Max' row is our answer - the most coins we can collect on the best part of the path!

In simpler terms, Kadane's algorithm keeps track of the current sum and the maximum sum as you traverse the path (or array). If the current sum ever drops below zero, we reset it. The maximum sum we've seen during our journey is our answer!

- [1186. Maximum Subarray Sum with One Deletion](https://leetcode.com/problems/maximum-subarray-sum-with-one-deletion/)
- [53. Maximum Subarray](https://leetcode.com/problems/maximum-subarray/)