---
layout: post
title: 190527 LeetCode(7) Reverse Integer
tags:
  - Algorithm
  - LeetCode
---

## 7. Reverse Integer

##### Description

<https://leetcode.com/problems/reverse-integer/>

Given a 32-bit signed integer, reverse digits of an integer.

**Example 1:**

```
Input: 123
Output: 321
```

**Note:**
Assume we are dealing with an environment which could only store integers within the 32-bit signed integer range: [−2^31,  2^31 − 1]. For the purpose of this problem, assume that your function returns 0 when the reversed integer overflows.



##### Solution

1. my first approach: using Stack

- make `x` as String and push an element into the stack.
  - If the last element is '-', `x` is negative number.  
- attach the pop element to a String.
- validity check and return result.

```java
class Solution {
    public int reverse(int x) {
        String tmp=Integer.toString(x);
        Stack<Character> st=new Stack<>();
        boolean isNegative=false;
        
        for(int i=0;i<tmp.length();i++)
            st.push(tmp.charAt(i));
        
        tmp="";
        
        while(!st.isEmpty())
            tmp+=st.pop();
        
        if(tmp.charAt(tmp.length()-1)=='-'){
            isNegative=true;
            tmp=tmp.substring(0,tmp.length()-1);
        }
            
        
        long result=Long.parseLong(tmp);
        if(isNegative)
            result=-result;
        
        if(result>Integer.MAX_VALUE || result<Integer.MIN_VALUE)
            return 0;
        else
            return (int)result;
    }
}
```

Runtime: **4 ms**

Memory Usage: **34.4 MB**



2. More efficient solution: using math

```java
class Solution {
    public int reverse(int x) {
        long result=0;
        while(x!=0){
            int tmp=x%10;
            x/=10;
            result=result*10+tmp;
        }
        
        if(result>Integer.MAX_VALUE || result<Integer.MIN_VALUE)
            return 0;
        else
            return (int)result;
    }
}
```

Runtime: **1 ms**

Memory Usage: **32.7 MB**