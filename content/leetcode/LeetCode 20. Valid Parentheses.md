---
title: "LeetCode 20. Valid Parentheses"
date: 2024-01-10T17:59:52+08:00
tags:
 - leetcode
draft: false
---

[Question Link](https://leetcode.com/problems/valid-parentheses/)  
### Description:
Given a string `s` containing just the characters `'('`, `')'`, `'{'`, `'}'`, `'['` and `']'`, determine if the input string is valid.

An input string is valid if:

1. Open brackets must be closed by the same type of brackets.
2. Open brackets must be closed in the correct order.
3. Every close bracket has a corresponding open bracket of the same type.

### Examples
#### Example 1
```text
Input: s = "()"
Output: true
```
#### Example 2
```
Input: s = "()[]{}"
Output: true
```
#### Example 3
```
Input: s = "(]"
Output: false
```
#### Constraints
- `1 <= s.length <= $10^4$`
- `s` consists of parentheses only `'()[]{}'`.

### Thoughts

### Solution
```python
class Solution:
    def isValid(self, s: str) -> bool:
        stack = []
        l_paren = ['(', '[', '{']
        r_paren = [')', ']', '}']

        for i in s:
            if i in l_paren:
                stack.append(i)
            if i in r_paren:
                if len(stack) == 0:
                    return False
                latest = stack.pop()
                if r_paren[l_paren.index(latest)] != i:
                    return False
        if len(stack) > 0:
            return False
        else:
            return True
```