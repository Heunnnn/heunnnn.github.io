---
layout: post
title: 190604 LeetCode(9) Palindrome Number
tags:
  - Algorithm
  - LeetCode
---

## 9. Palindrome Number

##### Description

<https://leetcode.com/problems/palindrome-number/>

Determine whether an integer is a palindrome. An integer is a palindrome when it reads the same backward as forward.

**Example 1:**

```
Input: 121
Output: true
```

**Follow up:**

Could you solve it without converting the integer to a string?



##### Solution

- 

```java
class Solution {
    public boolean isPalindrome(int x) {
        int ref=x;
        if(x<0)
            return false;
        
        int result=0;
        while(x!=0){
            result=result*10+(x%10);
            x/=10;
        }
        
        if(ref==result)
            return true;
        else
            return false;
    }
}
```

Runtime: **6 ms**

Memory Usage: **35.1 MB**