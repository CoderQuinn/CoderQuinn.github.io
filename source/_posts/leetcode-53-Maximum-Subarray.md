---
title: leetcode 53 Maximum Subarray
top: false
cover: false
toc: true
mathjax: true
date: 2020-08-14 19:36:21
password:
summary:
tags: leetcode
categories: 数据结构与算法
---

# leetcode 53 Maximum Subarray 最大子序和

## 蛮力枚举法

很容易想到的一个算法，就是逐一枚举出所有子列和进行比较，找出最大值，其代码实现如下：

```c++
    int maxSubArray(vector<int>& nums) {
        int result = INT_MIN;
        size_t size = nums.size();
        for (int i = 0; i < size; ++i) {
            for (int j = i; j < size; ++j) {
                int sum = 0;
                for (int k = i; k <= j; ++k) {
                    sum += nums[k];
                }
                result = max(result, sum);
            }
        }
        
        return result;
    }
```

注意到，蛮力枚举法的时间复杂度是O(n<sup>3</sup>)，显然并不是一个实用的算法。

## 优化枚举法

注意观察蛮力枚举法的过程，我们发现其中[i...j]的子序列和，完全可以由[i...j-1]的结果再累加nums[j]求出，没必要再从头计算。于是有了改进的算法：

```c++
    int maxSubArray(vector<int>& nums) {
        int result = INT_MIN;
        size_t size = nums.size();
        for (int i = 0; i < size; ++i) {
            int sum = 0;
            for (int j = i; j < size; ++j) {
                sum += nums[j];
                result = max(result, sum);
            }
        }
        
        return result;
    }
```

优化枚举法的时间复杂度是O(n<sup>2</sup>)，仍然不是一个最快的算法。

## 分治法

基本思想就是将原问题拆分为若干子问题，分别解决后再将结果合而治之，多用递归调用的形式实现

1. 将[𝟏. . 𝒏]序列分为[𝟏. . 𝒏/𝟐]和[𝒏/𝟐 + 𝟏. . ] 
2. 递归求解子问题 
   
   序列[𝟏. . 𝒏/𝟐] 的最大子序和 s<sub>1</sub>
   
   序列[𝒏/𝟐 + 𝟏. . 𝒏] 的最大子序和s<sub>2</sub>
   
   跨中点的最大子序列和 s<sub>3</sub>
3. 合并子问题的解，得到s<sub>max</sub> = 𝒎𝒂𝒙{  s<sub>1</sub>, s<sub>2</sub>, s<sub>3</sub>}

代码实现如下:

```c++
    int maxSubArray(vector<int>& nums) {
        return divide_conquer(nums, 0, nums.size() - 1);
    }
    
    int divide_conquer(vector<int>& nums, long left, long right) {
        if (left == right) return nums[left];
        
        long mid = (left + right) / 2;
        int left_max = divide_conquer(nums, left, mid);
        int right_max = divide_conquer(nums, mid + 1, right);
        int crossing_middle_max = crossing_middle_sub_max(nums, left, mid, right);
        return max(max(left_max, right_max), crossing_middle_max);
    }
    
    int crossing_middle_sub_max(vector<int>& nums, long left, long mid, long right) {
        int left_sum = INT_MIN;
        int right_sum = INT_MIN;
        int sum = 0;
        for (long i = mid; i >= left; i--) {
            sum += nums[i];
            left_sum = max(sum, left_sum);
        }
        
        sum = 0;
        for (long i = mid+1; i <= right; i++) {
            sum += nums[i];
            right_sum = max(sum, right_sum);
        }
        return left_sum + right_sum;
    }
```

其递归式为T(N) = 2 T(N/2) + O(N)，由递归树或主定理很容易求得其时间复杂度为T(N) = O(NLogN)。