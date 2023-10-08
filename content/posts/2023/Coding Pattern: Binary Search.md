---
title: "Coding Pattern: Binary Search"
date: 2023-10-08T16:28:21-04:00
draft:
categories: 
    - Programming
tags: 
    - Algorithm
    - LeetCode
    
---

# Overview

In one word, binary search is to search for a target in a sorted array. The idea is to shrink the search space to empty.

It must be sorted because we can be sure how to shrink the search space and we normally reduce the search space by half so the time complexity is `O(logN)` where `N` is the size of entire search space.

One common problem to understand *Binary Search* is how to identify the boundary of the search space. 

The below is the pattern I am using to solve the most *Binary Search* questions.

```python
class Solution:
	def search(self, nums: List[int], target: int) -> int:
		# The boundary is inclusive which determines the while condition
		head, tail = 0, len(nums) - 1
		# since the boundary is inclusive on the two ends of the nums array
		# we should search even when head == tail and we should just check 
		# that particular value
		while head <= tail:
			# we use floor division to make sure mid is an integer otherwise
			# it cannot be used to index
			# and using floor division will always result in a smaller value
			# towards head so we don't want to stop search when head < tail 
			mid = (head + tail) // 2
			# this is the part I love about this approach
			# we can return the result right away if it is equal to the target as
			# 1: we don't have to go through all search space
			# 2: we can safely eliminate mid point in the next search
			if nums[mid] == target:
				return mid
			# if target is larger than nums[mid], it indicates the result sits
			# on the left of mid, and otherwise the result sits on the right
			if nums[mid] < target:
				# since we are sure nums[mid] is not the result, we don't have
				# to check nums[mid] again and we could move head to mid + 1
				# otherwise, we move tail to mid - 1
				head = mid + 1
			else:
				tail = mid - 1
		# we are sure no results are found since all space are searched
		return -1
```

# Example Questions
[69. Sqrt(x)](https://leetcode.com/problems/sqrtx/)

```ad-question
Given a non-negative integer `x`, return _the square root of_ `x` _rounded down to the nearest integer_. The returned integer should be **non-negative** as well.

You **must not use** any built-in exponent function or operator.

- For example, do not use `pow(x, 0.5)` in c++ or `x ** 0.5` in python.

**Example 1:**

**Input:** x = 4
**Output:** 2
**Explanation:** The square root of 4 is 2, so we return 2.

**Example 2:**

**Input:** x = 8
**Output:** 2
**Explanation:** The square root of 8 is 2.82842..., and since we round it down to the nearest integer, 2 is returned.

**Constraints:**

- `0 <= x <= 231 - 1`
```

Solution:

```python
class Solution:
	# we are using binary search because the search space is sorted and must be
	# from [1, x]
    def mySqrt(self, x: int) -> int:
        if x < 2: return x
        # The search space is [0, x//2] since it is to find the square root
        head, tail = 0, x // 2
	
        while head <= tail:
            mid = (head + tail) // 2
            temp = mid * mid
            if temp == x:
                return mid
            elif temp < x:
                head = mid + 1
            else:
                tail = mid - 1
        
        if mid * mid > x:
            return mid - 1
        else:
            return mid
```


[33. Search in Rotated Sorted Array](https://leetcode.com/problems/search-in-rotated-sorted-array/)

```ad-question
There is an integer array `nums` sorted in ascending order (with **distinct** values).

Prior to being passed to your function, `nums` is **possibly rotated** at an unknown pivot index `k` (`1 <= k < nums.length`) such that the resulting array is `[nums[k], nums[k+1], ..., nums[n-1], nums[0], nums[1], ..., nums[k-1]]` (**0-indexed**). For example, `[0,1,2,4,5,6,7]` might be rotated at pivot index `3` and become `[4,5,6,7,0,1,2]`.

Given the array `nums` **after** the possible rotation and an integer `target`, return _the index of_ `target` _if it is in_ `nums`_, or_ `-1` _if it is not in_ `nums`.

You must write an algorithm with `O(log n)` runtime complexity.

**Example 1:**

**Input:** nums = [4,5,6,7,0,1,2], target = 0
**Output:** 4

**Example 2:**

**Input:** nums = [4,5,6,7,0,1,2], target = 3
**Output:** -1

**Example 3:**

**Input:** nums = [1], target = 0
**Output:** -1

**Constraints:**

- `1 <= nums.length <= 5000`
- `-104 <= nums[i] <= 104`
- All values of `nums` are **unique**.
- `nums` is an ascending array that is possibly rotated.
- `-104 <= target <= 104`
```

Solution

```python
class Solution:
    def search(self, nums: list[int], target: int) -> int:
        head = 0
        tail = len(nums) - 1

        while head <= tail:
            mid = (head + tail) // 2
            if nums[mid] == target:
                return mid
            if nums[head] <= nums[mid]:
                if nums[head] <= target <= nums[mid]:
                    tail = mid - 1
                else:
                    head = mid + 1
            else:
                if nums[mid] <= target <= nums[tail]:
                    head = mid + 1
                else:
                    tail = mid - 1
        
        return -1
```

[4. Median of Two Sorted Arrays](https://leetcode.com/problems/median-of-two-sorted-arrays/)

```ad-question
Given two sorted arrays `nums1` and `nums2` of size `m` and `n` respectively, return **the median** of the two sorted arrays.

The overall run time complexity should be `O(log (m+n))`.

**Example 1:**

**Input:** nums1 = [1,3], nums2 = [2]
**Output:** 2.00000
**Explanation:** merged array = [1,2,3] and median is 2.

**Example 2:**

**Input:** nums1 = [1,2], nums2 = [3,4]
**Output:** 2.50000
**Explanation:** merged array = [1,2,3,4] and median is (2 + 3) / 2 = 2.5.

**Constraints:**

- `nums1.length == m`
- `nums2.length == n`
- `0 <= m <= 1000`
- `0 <= n <= 1000`
- `1 <= m + n <= 2000`
- `-106 <= nums1[i], nums2[i] <= 106`
```

Solution:

This question has the following information:
1. two **sorted** arrays
2. find the median of the two sorted arrays which indicate the non-descending order
3. time complexity is `O(log(m+n))` which is a big sign of *Binary Search*

The idea is to find the slice, say `mid1` and `mid2`, in two arrays and `nums1[mid1] < nums2[mid2]` and `nums2[mid2] < nums1[mid1]` so we know the elements on the left of `mid1` and `mid2` are smaller than the elements on the other side. Then we could get the median based on the size of the total elements.

And one key in this question is to make sure all the elements on the left add up should be equal to the half size of entire elements, then we can ensure the `mid1` and `mid2` represent the position of median values.

```python
class Solution:
    def findMedianSortedArrays(self, nums1: List[int], nums2: List[int]) -> float:
        if len(nums1) > len(nums2):
            return self.findMedianSortedArrays(nums2, nums1)
        m, n = len(nums1), len(nums2)
        # the search space is the length of the array unlike the index in above questions, this is important since the array is 0-indexed.
        head, tail = 0, m
        while head <= tail:
	        # this the length we attempt to grap from nums1
            mid1 = (head + tail) // 2
			# this is the length we attempt to grap from nums2, it is (m + n + 1)//2 - mid1 so mid2 will not be negative
			# when mid1 increases and equal to m, (m + n)//2 - m could be negative when n < m and (m + n) is odd, thus (m + n + 1) ensure mid2 won't be negative
            mid2 = (m + n + 1) // 2 - mid1
			# if the length is 0 for either nums1 or nums2, the value on the left should be infinite negative since there is no value
            maxLeft1 = float("-inf") if mid1 == 0 else nums1[mid1 - 1]
            maxLeft2 = float("-inf") if mid2 == 0 else nums2[mid2 - 1]
			# likewise, if entire elements are grapped, the element on the right is infinite positive
            minRight1 = float("inf") if mid1 == m else nums1[mid1]
            minRight2 = float("inf") if mid2 == n else nums2[mid2]
			# since we get mid2 from mid1 and the total length, the mid1 and mid2 should always slice the entire elements into half and we only need check if the elements to the left of mid1 and mid2 are less than those on the other side.
            if maxLeft1 <= minRight2 and maxLeft2 <= minRight1:
	            # if so, we calculate the median given the total size
                if (m + n) % 2 == 0:
	                # if it is even, we need the value on the left and right of mid since it requires two value
                    return (min(minRight1, minRight2) + max(maxLeft1, maxLeft2)) / 2
                else:
	                # otherwise, we only need the max of maxLeft1 and maxLeft2 
                    return max(maxLeft1, maxLeft2)
            else:
	            # if the ending condition doesn't meet, we should adjust the search space to contain less elements from nums1 since the left value of mid1, maxLeft1, is larger than the right value of mid2, minRight2
                if maxLeft1 > minRight2:
                    tail = mid1 - 1
                else:
                    head = mid1 + 1

```


# Takeaways
```ad-hint
In my opinion, *Binary Search* is a possible solution when the questions contain the following information:
1. The question asks us to find an result for a given condition
2. The search space can be sorted based on the given condition

In this case, as long as we can transform the question into the case above, we could use *Binary Search* to solve it.
```