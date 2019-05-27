---
layout: post
title: 190525 LeetCode(1) Two Sum
tags:
  - Algorithm
  - LeetCode
  - Brute-Force
  - HashMap
---

## 1. Two Sum

##### Description

<https://leetcode.com/problems/two-sum/>

Given an array of integers, return **indices** of the two numbers such that they add up to a specific target.

You may assume that each input would have **exactly** one solution, and you may not use the *same* element twice.

**Example:**

```
Given nums = [2, 7, 11, 15], target = 9,

Because nums[0] + nums[1] = 2 + 7 = 9,
return [0, 1].
```



##### Solution

1. Using Brute-Force

- We can try everything possible.
- Loop through each element `nums[i]` and find if there is another value that equals to `target-nums[i]`
- efficiency
  - time complexity: `O(n)`
  - space complexity: `O(1)`

```java
class Solution {
    public int[] twoSum(int[] nums, int target) {
        int[] solution={0,0};
        boolean isFound=false;
        for(int i=0;i<nums.length-1;i++){
            int tmp=target-nums[i];
            for(int j=i+1;j<nums.length;j++){
                if(nums[j]==tmp){
                    solution[0]=i;
                    solution[1]=j;
                    isFound=true;
                    break;
                }
            }   
            if(isFound)
                break;
        }
        return solution;
    }
}
```

Runtime: **14 ms**

Memory Usage: **36.5 MB**



2. Using HashMap

- In the first iteration, we add each element's value and its index to the table.
- Then, in the second iteration we check if each element's complement `target - nums[i]` exists in the table.

```java
class Solution {
    public int[] twoSum(int[] nums, int target) {
        int[] solution={0,0};
        HashMap<Integer, Integer> map=new HashMap<>();
        for(int i=0;i<nums.length;i++)
            map.put(nums[i],i);
        
        for(int i=0;i<nums.length;i++){
            int tmp=target-nums[i];
            if(map.containsKey(tmp) && map.get(tmp)!=i){
                solution[0]=i;
                solution[1]=map.get(tmp);
                break;
            }
        }
        
        return solution;
    }
}
```

Runtime: **2 ms**

Memory Usage: **37.3 MB**