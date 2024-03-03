---
title: "LeetCode 55. Jump Game"
date: 2023-05-23T10:38:33+08:00
tags:
 - leetcode
draft: false
---

[Question Link](https://leetcode.com/problems/jump-game/description/)  
### Description:
You are given an integer array `nums`. You are initially positioned at the array's **first index**, and each element in the array represents your maximum jump length at that position.

*Return `true` if you can reach the last index, or `false` otherwise.*

### Examples
#### Example 1
```text
Input: nums = [2,3,1,1,4]
Output: true
Explanation: Jump 1 step from index 0 to 1, then 3 steps to the last index.
```
#### Example 2
```
Input: nums = [3,2,1,0,4]
Output: false
Explanation: You will always arrive at index 3 no matter what. Its maximum jump length is 0, which makes it impossible to reach the last index.
```
### Thoughts
1. Use a *maximum* variable to store the farthest index I can reach currently.
2. Traverse *nums* and check if $maximum ≧ i$. Indicate whether the jump method can reach the current index.

### 解法思路
1. 用一個*maximum*存目前可以到達最遠的idx
2. 遍歷*nums*, 檢查*maximum*是否大於等於i, 表示jump的方法是否可以到達目前的index.

### Solution
```cpp
#include <limits.h>
class Solution {
public:
    bool canJump(vector<int>& nums) {
        int maximum = nums[0];
        for (int i=1; i<nums.size(); ++i){
            if (maximum < i)
                return false;
            maximum = max(maximum, i+nums[i]);
        }
        return true;
    }
};
```

> O(n)Runtime beats 89.98%, memory usage beats 94.33%