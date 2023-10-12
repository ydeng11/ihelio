---
title: "Coding Pattern: Two Pointers"
date: 2023-10-12T00:44:00-04:00
draft: 
categories: 
    - Programming
tags: 
    - Algorithm
    - LeetCode
---

### Overview

It is not easy to summarize the pattern of **Two Pointers**, but most likely it is used for list and linked list and the required time complexity is `O(N)` - the underlying pattern allows us to use **Two Pointers** to go through the list once to get the results.

Common usage:
1. **Linear Structure**:
    - Typically applied to a sorted array or linked list.
    - Two pointers might move in the same direction or in opposite directions.
2. **Classic Patterns**:
    a. **Converging Pointers** (often used in sorted arrays): - Start one pointer at the beginning (`left`) and another at the end (`right`). - Move them toward each other until they meet or until some condition is satisfied. - Example: Checking if a sorted array has two numbers that sum up to a target.
    b. **Sliding Window**: - Use two pointers to represent the start and end of a window, then "slide" the window through the array/sequence. - Example: Finding the longest substring without repeating characters.
    c. **Fast and Slow Pointers** (often used in linked lists): - One pointer moves twice (or more times) as fast as the other. - Useful for detecting cycles in a linked list (Floyd's Cycle Detection Algorithm) or finding the middle element.
3. **Usage Scenarios**:
    - **Finding a Pair with a Given Sum/Target**: Given a sorted array, determine if there's a pair that sums up to a target. Move the `left` and `right` pointers based on the sum comparison to the target.
    - **Removing Duplicates**: Two pointers can be used to remove duplicates from a sorted array or linked list, with one pointer iterating through and another pointing to the last non-duplicate item.
    - **Palindrome Checking**: To determine if a string or linked list is a palindrome, you can use two pointers moving from the two ends towards the center.
    - **Max/Min Subarray/Sublist**: Using the sliding window variant, find the subarray with the maximum/minimum sum or other properties.
4. **Advantages**:
    - **Efficiency**: The two-pointer technique can sometimes convert a brute force solution with time complexity O(n^2) to a more efficient O(n) solution.
    - **Space**: This method is in-place and typically uses O(1) extra space.
### Example Questions

#### [88. Merge Sorted Array](https://leetcode.com/problems/merge-sorted-array/)

You are given two integer arrays `nums1` and `nums2`, sorted in **non-decreasing order**, and two integers `m` and `n`, representing the number of elements in `nums1` and `nums2` respectively.

**Merge** `nums1` and `nums2` into a single array sorted in **non-decreasing order**.

The final sorted array should not be returned by the function, but instead be _stored inside the array_ `nums1`. To accommodate this, `nums1` has a length of `m + n`, where the first `m` elements denote the elements that should be merged, and the last `n` elements are set to `0` and should be ignored. `nums2` has a length of `n`.

**Example 1:**

**Input:** nums1 = [1,2,3,0,0,0], m = 3, nums2 = [2,5,6], n = 3
**Output:** [1,2,2,3,5,6]
**Explanation:** The arrays we are merging are [1,2,3] and [2,5,6].
The result of the merge is [1,2,2,3,5,6] with the underlined elements coming from nums1.

**Example 2:**

**Input:** nums1 = [1], m = 1, nums2 = [], n = 0
**Output:** [1]
**Explanation:** The arrays we are merging are [1] and [].
The result of the merge is [1].

**Example 3:**

**Input:** nums1 = [0], m = 0, nums2 = [1], n = 1
**Output:** [1]
**Explanation:** The arrays we are merging are [] and [1].
The result of the merge is [1].
Note that because m = 0, there are no elements in nums1. The 0 is only there to ensure the merge result can fit in nums1.

**Constraints:**

- `nums1.length == m + n`
- `nums2.length == n`
- `0 <= m, n <= 200`
- `1 <= m + n <= 200`
- `-109 <= nums1[i], nums2[j] <= 109`

**Follow up:** Can you come up with an algorithm that runs in `O(m + n)` time?

Solution:

They are sorted arrays, and the follow-up asks if it can be done in linear time. And the question is to merge two sorted arrays into one sorted array. Intuitively we could do it zigzag. 

```python
class Solution:
    def merge(self, nums1: List[int], m: int, nums2: List[int], n: int) -> None:
        """
        Do not return anything, modify nums1 in-place instead.
        """
        if n == 0: return
	    # copy nums1 since it is larger
        nums1_cp = nums1[:]
		# create three pointers p1 and p2 starting at left end
        p1, p2, p3 = 0, 0, 0
		# when p1 and p2 are less than the size of nums1 and nums2, we continue to merge
		# we move p3 every iteration since we are inserting the value
        while p1 < m or p2 < n:
            if p2 == n:
                nums1[p3] = nums1_cp[p1]
                p1 += 1
            elif p1 == m:
                nums1[p3] = nums2[p2]
                p2 += 1
            elif nums1_cp[p1] <= nums2[p2]:
                nums1[p3] = nums1_cp[p1]
                p1 += 1 
            else:
                nums1[p3] = nums2[p2]
                p2 += 1
            p3 += 1


```

#### [11. Container With Most Water](https://leetcode.com/problems/container-with-most-water/)

You are given an integer array `height` of length `n`. There are `n` vertical lines drawn such that the two endpoints of the `ith` line are `(i, 0)` and `(i, height[i])`.

Find two lines that together with the x-axis form a container, such that the container contains the most water.

Return _the maximum amount of water a container can store_.

**Notice** that you may not slant the container.

**Example 1:**

![](https://s3-lc-upload.s3.amazonaws.com/uploads/2018/07/17/question_11.jpg)

**Input:** height = [1,8,6,2,5,4,8,3,7]
**Output:** 49
**Explanation:** The above vertical lines are represented by array [1,8,6,2,5,4,8,3,7]. In this case, the max area of water (blue section) the container can contain is 49.

**Example 2:**

**Input:** height = [1,1]
**Output:** 1

**Constraints:**

- `n == height.length`
- `2 <= n <= 105`
- `0 <= height[i] <= 104`

Solution

This question asks the max size of the container and we know the size is determined by the shorter column between two end. In other words, if `height[left]` is less than `height[right]`, the size could is at most `height[left]*(right - left)` no matter how we move `height[right]`

Therefore, we could try to find a larger `height[left]` by moving `left` and check it yields a larger size. In this case, we could use **Two Pointers** starting at the left and right end of `height`. 

```python
class Solution:
    def maxArea(self, height: List[int]) -> int:
	    # create two pointers at left and right end
        head = 0
        tail = len(height) - 1
        res = 0
        # continue to check when two pointers are not met yet
        while head < tail:
            container_len = tail - head
            # update the max size found
            res = max(res, container_len * min(height[head], height[tail]))
            # move tail if height[tail] is less and otherwise move head
            if height[head] > height[tail]:
                tail -= 1
            else:
                head += 1
        return res
```

#### [42. Trapping Rain Water](https://leetcode.com/problems/trapping-rain-water/)

Given `n` non-negative integers representing an elevation map where the width of each bar is `1`, compute how much water it can trap after raining.

**Example 1:**

![](https://assets.leetcode.com/uploads/2018/10/22/rainwatertrap.png)

**Input:** height = [0,1,0,2,1,0,1,3,2,1,2,1]
**Output:** 6
**Explanation:** The above elevation map (black section) is represented by array [0,1,0,2,1,0,1,3,2,1,2,1]. In this case, 6 units of rain water (blue section) are being trapped.

**Example 2:**

**Input:** height = [4,2,0,3,2,5]
**Output:** 9

**Constraints:**

- `n == height.length`
- `1 <= n <= 2 * 104`
- `0 <= height[i] <= 105`

Solution:

The idea is to calculate the trapped water using the $min(left_{max}, right_{max})$ as the water will be trapped by the shorter block like [11. Container With Most Water](#11. Container With Most Water ) except we don't move the higher block here.  
Like the dynamic programming, we will update the $left_{max}$ and $right_{max}$ but we don’t have to store all heights.
The reason we need only one pass because the $left_{max}$ and $right_{max}$ will only increase when we move their pointers. If $left_i{max}<right_j{max}$ at point $i$, and $j$, we should expect $left_i{max}<right_i{max}$.

```python
class Solution:
    def trap(self, height: List[int]) -> int:
        l, r = 0, len(height) - 1
        l_h, r_h = height[0], height[-1]
        out = 0
        while l < r:
			# update the max height from left and right
            l_h = max(l_h, height[l])
            r_h = max(r_h, height[r])
            # if left max height is less than right max height
			# we should move left pointer and calculate the trapped water at l
            if l_h < r_h:
                out += l_h - height[l]
                l += 1
            else:
                out += r_h - height[r]
                r -= 1
        return out

```

#### LinkedList

As one kind of list, **Two Pointers** could also be used for linked list, though sometimes we call it **Two Runners**. The idea is to have a slow and a fast runner. Normally when fast runner finishes, the slow runner is at the middle of the linked list which is useful.

##### [876. Middle of the Linked List](https://leetcode.com/problems/middle-of-the-linked-list/)

Given the `head` of a singly linked list, return _the middle node of the linked list_.

If there are two middle nodes, return **the second middle** node.

**Example 1:**

![](https://assets.leetcode.com/uploads/2021/07/23/lc-midlist1.jpg)

**Input:** head = [1,2,3,4,5]
**Output:** [3,4,5]
**Explanation:** The middle node of the list is node 3.

**Example 2:**

![](https://assets.leetcode.com/uploads/2021/07/23/lc-midlist2.jpg)

**Input:** head = [1,2,3,4,5,6]
**Output:** [4,5,6]
**Explanation:** Since the list has two middle nodes with values 3 and 4, we return the second one.

**Constraints:**

- The number of nodes in the list is in the range `[1, 100]`.
- `1 <= Node.val <= 100`

Solution:
- When length is odd, the fast runner will finish at the end of the list and the slow runner is at the middle point
- When length is even, the fast runner will finish right after the end of the list and the slow runner is at the start of the second half list

```python
# Definition for singly-linked list.
# class ListNode:
#     def __init__(self, val=0, next=None):
#         self.val = val
#         self.next = next
class Solution:
    def middleNode(self, head: Optional[ListNode]) -> Optional[ListNode]:
        slow, fast = head, head
        while fast and fast.next:
            slow = slow.next
            fast = fast.next.next
        return slow
```
