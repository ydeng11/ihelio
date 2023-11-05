---
title: "Coding Pattern: Recursion"
date: 2023-11-05T16:41:32-05:00
draft: 
categories: 
    - Programming
tags: 
    - Algorithm
    - LeetCode
---
# Overview

Recursion is a way of solving a problem by breaking it down into smaller problems of the same type. The smaller problems are then solved recursively, until a base case is reached. The base case is a simple problem that can be solved without recursion.

Imagine you have a big box of toy blocks, and each block represents a problem you need to solve. Some blocks are big (complex problems), and some are tiny (simple problems). To solve a big problem, you take apart the big block to find smaller blocks inside, and then, if needed, you take apart those smaller blocks to find even tinier blocks inside them, and so on, until you reach the tiniest blocks that you can easily understand and solve.

Recursion is like this process of opening boxes to find smaller boxes inside, until you reach the smallest box which you can solve easily. Once you solve the smallest problem, you use its solution to solve the bigger problem it came from, and you keep doing this until you've solved the original big problem.

**Figure:**

```python
Big Block (Big Problem)
|
|____ Medium Block (Medium Problem)
|    |
|    |____ Small Block (Simple Problem)
|    |
|    |____ Solved!
|
|____ Solved using solution from Medium Block!
|
|____ Big Problem Solved!

```

In this illustration, you started with a big problem (Big Block). You broke it down into a medium problem (Medium Block), and then into a simple problem (Small Block). Once the simple problem was solved, you used its solution to solve the medium problem, and then the big problem. That's how recursion works!

# Common places to use recursion

## Bottom-Up Dynamic Programming

**Explanation**: A method to solve complex problems by breaking them down into simpler subproblems. Unlike top-down, which starts with the main problem and divides it, bottom-up solves the smallest subproblems first and builds up to the main problem.

**Figure**:

```python
Fib(5)
  /   \
Fib(4) Fib(3)
      /     \
   Fib(3)  Fib(2)  ... and so on.
```

Using bottom-up, we'd start with the smallest calculations (Fib(0), Fib(1)) and use their solutions to build up to Fib(5).

**Common Problems**:
  - [Climbing Stairs](https://leetcode.com/problems/climbing-stairs/)
  - [Coin Change](https://leetcode.com/problems/coin-change/)
  - [House Robber](https://leetcode.com/problems/house-robber/)

## Backtracking

**Explanation**: A general algorithm for finding all (or some) solutions to computational problems by incrementally building candidates towards solutions and abandoning a candidate as soon as it is determined to be non-solution.

**Figure**:

```sql
Start
  |
Option1 ---- Option2 ---- Option3 (Backtrack)
  |   
Option2a      ... and so on.
```

If a solution isn’t possible, we backtrack to try the next possibility.
**Common Problems**:

- [Permutations](https://leetcode.com/problems/permutations/)
- [N-Queens](https://leetcode.com/problems/n-queens/)
- [Combination Sum](https://leetcode.com/problems/combination-sum/)

## Depth-First Search (DFS)

   **Explanation**: An algorithm for traversing trees or graphs by exploring as far as possible along each branch before backtracking.

   **Figure**:

   ```mathmatica
     A
    / \
   B   C
   |   | \
   D   E  F

   ```

   DFS traversal: A, B, D, C, E, F

   **Common Problems**:

   - [Max Area of Island](https://leetcode.com/problems/max-area-of-island/)
   - [Number of Islands](https://leetcode.com/problems/number-of-islands/)
   - [All Paths From Source to Target](https://leetcode.com/problems/all-paths-from-source-to-target/)

## Divide and Conquer

   **Explanation**: A method to break a problem into smaller subproblems that are similar to the original problem, solve these subproblems recursively, and then combine the solutions to get a solution to the original problem.

   **Figure**:

   ```
     problem
    /   |    \
   sub1 sub2  sub3
   ```

   Solve Sub1, Sub2, and Sub3, then combine them for the solution.

   **Common Problems**:

   - [Merge Sort](https://leetcode.com/problems/sort-an-array/)
   - [Maximum Subarray](https://leetcode.com/problems/maximum-subarray/)
   - [Count of Smaller Numbers After Self](https://leetcode.com/problems/count-of-smaller-numbers-after-self/)

## Preorder, Inorder, and Postorder Tree Traversals

   **Explanation**: Systematic ways to traverse all nodes in a tree. Preorder visits the current node before its children. Inorder visits the left child, then the current node, then the right child. Postorder visits the children before the current node (see [[Tree & Graph#Tree]]).

   **Figure**:

   ```css
     A
    / \
   B   C
   ```

   Preorder: A, B, C

   Inorder: B, A, C

   Postorder: B, C, A

   **Common Problems**:

   - [Binary Tree Inorder Traversal](https://leetcode.com/problems/binary-tree-inorder-traversal/)
   - [Construct Binary Tree from Preorder and Inorder Traversal](https://leetcode.com/problems/construct-binary-tree-from-preorder-and-inorder-traversal/)
   - [Populating Next Right Pointers in Each Node](https://leetcode.com/problems/populating-next-right-pointers-in-each-node/)

# Complexity analysis

## Time complexity

Normally the time complexity is proportional to the entire search space. For instance, the tree traversals would go through every node once so the time complexity is the size of the tree.

## Space complexity

The major space complexity is from the recursion stack. Thus we need analyze the deepest depth the recursion could reach. If we are using DFS on a tree, we normally would return when DFS reaches the leaf node thus the recursion stack is the height of the tree at most.
