---
title: leetcode算法，查找指定数和的元素的索引
date: 2018-04-04 13:57:23
categories: 
	- 算法
tags:
	- 数组
	- 查找index
---

### 问题描述
 **给定一个整数数列，找出其中和为特定值的那两个数。  
 你可以假设每个输入都只会有一种答案，同样的元素不能被重用。**

### 示例:   
	给定：nums = [2, 7, 11, 15] , target = 9  
	因为：nums[0] + nums[1] = 2 + 7 = 9 
	所以返回： [0, 1]

### 实现：
```java
public static int[] code(int [] arr,int num){
        Map<Integer,Integer> dataIndex = new HashMap<Integer, Integer>();

        for(int i=0,s=arr.length;i<s;i++){
            int v= num - arr[i]; //如果和相等，即代表减法之中，肯定有一个能找到对应的

            if (v >= 0) {//默认没有负数
                if(i != 0 && dataIndex.containsKey(v)){ //将前面存储的找到。即为相应的
                    return new int[]{dataIndex.get(v),i};
                }
                dataIndex.put(arr[i], i);//存储值和索引
            }

        }
        return null;
}
```
### leetcode最优解：
支持负数元素，少了判断

```
public int[] twoSum(int[] nums, int target) {
    Map<Integer, Integer> map = new HashMap<>();
    for (int i = 0; i < nums.length; i++) {
        int complement = target - nums[i];
        if (map.containsKey(complement)) {
            return new int[] { map.get(complement), i };
        }
        map.put(nums[i], i);
    }
    throw new IllegalArgumentException("No two sum solution");
}
```











