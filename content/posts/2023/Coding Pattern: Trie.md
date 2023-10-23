---
title: "Coding Pattern: Trie"
date: 2023-10-22T21:41:02-04:00
draft:
categories: 
    - Programming
tags: 
    - Algorithm
    - LeetCode
---
# Overview

The union-find algorithm is a data structure and algorithm that maintains a collection of disjoint sets. A disjoint set is a set of elements that are not connected to each other. The union-find algorithm can be used to perform the following operations:

- **Find:** Find the set that an element belongs to.
- **Union:** Merge two sets together.

The union-find algorithm is often used to solve problems that involve graph connectivity. For example, the union-find algorithm can be used to determine whether two nodes in a graph are connected, or to find all of the connected components in a graph.

Here is an illustrative example of the union-find algorithm:

Imagine that we have a set of 5 elements: A, B, C, D, and E. We can use the union-find algorithm to maintain a collection of disjoint sets of these elements.

Initially, all of the elements are in their own separate sets:

```
Set 1: A
Set 2: B
Set 3: C
Set 4: D
Set 5: E
```

Now, let's say that we want to merge Set 1 and Set 2. We can use the union-find algorithm to do this by combining the two sets into a single set:

```
Set 1: A, B
Set 3: C
Set 4: D
Set 5: E
```

We can also use the union-find algorithm to find the set that an element belongs to. For example, if we want to find the set that element B belongs to, we can use the union-find algorithm to determine that B is in Set 1.

The union-find algorithm is a powerful tool that can be used to solve a wide variety of problems. It is often used to solve problems that involve graph connectivity, such as finding the connected components in a graph or determining whether two nodes in a graph are connected.

Here is a diagram that illustrates the union-find algorithm:

```
        Set 1
       /    \
      A      B
       \    /
        Set 2
```

In this diagram, Set 1 and Set 2 are merged together. The union-find algorithm maintains a tree structure for each set. The root of the tree is the representative of the set. In this diagram, A is the representative of Set 1 and B is the representative of Set 2.

To find the set that an element belongs to, the union-find algorithm follows the pointers from the element to the root of the tree. For example, to find the set that B belongs to, we would follow the pointer from B to A. Since A is the root of the tree, we know that B is in Set 1.

To merge two sets together, the union-find algorithm makes the root of one tree the child of the root of the other tree. For example, to merge Set 1 and Set 2, we would make B the child of A. This would create a single tree that represents both Set 1 and Set 2.

The union-find algorithm is a very efficient algorithm for maintaining a collection of disjoint sets. It is often used in graph algorithms, such as Kruskal's algorithm and Prim's algorithm.

## Example of Implementation

```python
class UnionFind:
    def __init__(self):
        self.parents = {}
        self.size = {}

    def insert(self, x):
        if x not in self.parents:
            self.parents[x] = x
            self.size[x] = 1

	# recursively find the uppermost parent node (path compression)
    def find(self, node):
        if self.parents[node] != node:
            self.parents[node] = self.find(self.parents[node])
        return self.parents[node]

	# merge two objects into one set
    def union(self, x, y):
	    # insert x and y into the parents and size dict
        self.insert(x)
        self.insert(y)
        # find the parent of x and y (union by rank)
        px, py = self.find(x), self.find(y)
        # they are already in the same set if their parents are the same
        if px == py:
            return
    
        # always make px the less parent
        if self.size[px] > self.size[py]:
            px, py = py, px
        # so the parent of px is py since px is less
        self.parents[px] = py
        # increase the size of py
        self.size[py] += self.size[px]
```

**Space Complexity**:
	The space complexity for the Union-Find data structure is $O(n)$, where n is the number of elements. This is because, typically, two arrays (or vectors) of size $n$ are maintained: one for parent pointers and another for ranks/sizes.

**Time Complexity**:
    Without optimizations, the operations can be quite slow. However, with the **Union by Rank** and **Path Compression** optimizations, the operations become much faster.

- **Union Operation (with Union by Rank)**: This operation ensures that when two sets are combined, the smaller set (in terms of rank) is attached to the root of the larger set. This keeps the tree relatively flat. This operation is generally $O(1)$ but can be considered $O(logn)$ in the worst case because in the worst case the height of the tree can grow up to $logn$.
- **Find Operation (with Path Compression)**: When performing a `find` operation to determine the root of a particular element, the algorithm traverses up the tree and, in the process, flattens the tree by making every node point directly to the root. This makes future `find` operations faster. The amortized time complexity of the `find` operation with path compression (and union by rank) is near $O(1)$. However, when considering sequences of operations, the inverse Ackermann function, $α(n)$, often comes into play, making the operations effectively constant time for all practical purposes.
- **Union (combined with Find)**: Since the union operation often involves two `find` operations (to get the roots of the two sets being unioned), its time complexity is also governed by the `find` operation. Thus, its amortized time complexity is also effectively constant for practical purposes, but academically it's often cited as $O(α(n))$.

In summary, with the Union by Rank and Path Compression optimizations, the Union-Find operations become nearly constant time, $O(α(n))$, where $α(n)$ is the inverse Ackermann function, which grows extremely slowly. For most practical purposes, **it's considered constant time.**

# Example Questions

## [721. Accounts Merge](https://leetcode.com/problems/accounts-merge/)

Given a list of `accounts` where each element `accounts[i]` is a list of strings, where the first element `accounts[i][0]` is a name, and the rest of the elements are **emails** representing emails of the account.

Now, we would like to merge these accounts. Two accounts definitely belong to the same person if there is some common email to both accounts. Note that even if two accounts have the same name, they may belong to different people as people could have the same name. A person can have any number of accounts initially, but all of their accounts definitely have the same name.

After merging the accounts, return the accounts in the following format: the first element of each account is the name, and the rest of the elements are emails **in sorted order**. The accounts themselves can be returned in **any order**.

**Example 1:**

**Input:** accounts = [["John","johnsmith@mail.com","john_newyork@mail.com"],["John","johnsmith@mail.com","john00@mail.com"],["Mary","mary@mail.com"],["John","johnnybravo@mail.com"]]
**Output:** [["John","john00@mail.com","john_newyork@mail.com","johnsmith@mail.com"],["Mary","mary@mail.com"],["John","johnnybravo@mail.com"]]
**Explanation:**
The first and second John's are the same person as they have the common email "johnsmith@mail.com".
The third John and Mary are different people as none of their email addresses are used by other accounts.
We could return these lists in any order, for example the answer [['Mary', 'mary@mail.com'], ['John', 'johnnybravo@mail.com'],
['John', 'john00@mail.com', 'john_newyork@mail.com', 'johnsmith@mail.com']] would still be accepted.

**Example 2:**

**Input:** accounts = [["Gabe","Gabe0@m.co","Gabe3@m.co","Gabe1@m.co"],["Kevin","Kevin3@m.co","Kevin5@m.co","Kevin0@m.co"],["Ethan","Ethan5@m.co","Ethan4@m.co","Ethan0@m.co"],["Hanzo","Hanzo3@m.co","Hanzo1@m.co","Hanzo0@m.co"],["Fern","Fern5@m.co","Fern1@m.co","Fern0@m.co"]]
**Output:** [["Ethan","Ethan0@m.co","Ethan4@m.co","Ethan5@m.co"],["Gabe","Gabe0@m.co","Gabe1@m.co","Gabe3@m.co"],["Hanzo","Hanzo0@m.co","Hanzo1@m.co","Hanzo3@m.co"],["Kevin","Kevin0@m.co","Kevin3@m.co","Kevin5@m.co"],["Fern","Fern0@m.co","Fern1@m.co","Fern5@m.co"]]

**Constraints:**

- `1 <= accounts.length <= 1000`
- `2 <= accounts[i].length <= 10`
- `1 <= accounts[i][j].length <= 30`
- `accounts[i][0]` consists of English letters.
- `accounts[i][j] (for j > 0)` is a valid email.

## Solution

```python
class UnionFind:
    def __init__(self):
        self.parents = {}
        self.size = {}
  
    def insert(self, x):
        if x not in self.parents:
            self.parents[x] = x
            self.size[x] = 1

    def find(self, node):
        if self.parents[node] != node:
            self.parents[node] = self.find(self.parents[node])
        return self.parents[node]

    def union(self, x, y):
        self.insert(x)
        self.insert(y)
        px, py = self.find(x), self.find(y)
        if px == py:
            return
        if self.size[px] > self.size[py]:
            px, py = py, px
    
        self.parents[px] = py
        self.size[py] += self.size[px]


class Solution:
    def accountsMerge(self, accounts: List[List[str]]) -> List[List[str]]:
        uf = UnionFind()

        name_map = {}

        for account in accounts:
            name, emails = account[0], account[1:]
            for email in emails:
                name_map[email] = name
                # merge all the emails with the first email
                uf.union(emails[0], email)
    
        out = defaultdict(list)
        for key in uf.parents.keys():
		    # group all emails based on their parent in union-find
            out[uf.find(key)].append(key)
    
        res = []
        for key, val in out.items():
            res.append([name_map[key]] + sorted(val))
        return res
```

Time Complexity: $O(n)$ where $n$ is the number of emails, it is $n$ because of union-by-rank
Space Complexity: $O(n)$
