---
title: 16. 3Sum Closest
sidebar_label: "16. 3Sum Closest"
description: LeetCode
last_update:
  date: 2024-1-11
keywords:
  - LeetCode  
sidebar_position: 1
---

### 題目

Given an integer array nums of length n and an integer target, find three integers in nums such that the sum is closest to target.
Return the sum of the three integers.
You may assume that each input would have exactly one solution.


```md
Example 1:
Input: nums = [-1,2,1,-4], target = 1
Output: 2
Explanation: The sum that is closest to the target is 2. (-1 + 2 + 1 = 2).


Example 2:
Input: nums = [0,0,0], target = 1
Output: 0
Explanation: The sum that is closest to the target is 0. (0 + 0 + 0 = 0).

```

### 題目翻譯

這一題的目的是，給你一個陣列，和一個目標值(target)，現在從陣列中找出三個值相加，最後回傳目標值最近的3個數字，


### python 解題


#### 範例一
先把整個陣列 sort
設定一個 result 值 = num[0] + num[1] + num[2]
接著跑回圈
for first_num in range(len(nums))

再來一樣放三根指針

second_num = i + 1
third_num = len(nums) - 1

如果今天 third_num > second_num 就停止 while 回圈
while second_num < third_num:

先加總三根指針的值 total = nums[first] + nums[second] + nums[third]

如果今天 total-target 的絕對值 < result-target 的絕對值

[-1,2,1,-4]
sort -> [-4,-1,1,2]
1 + 2 + 3
result =|-4-1+1| = 4 -> 
1 + 2 + -1
total = |-4-1+2| = 3

這題的核心是三根指針的持續變動，三根指針持續變動加總值會一直變，所以只要找到一直變動後，跟目標值最近的加總指針就可以
如果從小到大，照順序增減的值加總 - 目標值 < 三根指針值加總 - 目標值，代表 result 太大，所以我們把 result 的值設定成指針為主的 total 值
4 - 1 = 3
3 - 1 = 2

ps. 一開始設定 result 的原因是因為要給三根指針一個基礎比較值

result = total = 3

再來判斷 total 跟目標值比較，如果 total < target 代表我們要把 第二根指針向右移動，這樣可以讓 total 變大，反之 total > target 我們要把第三根指針向左移動，如果 total = target 那直接回傳 result 就好
因為 total 目前是 3 > target 1
因此第三根指針往前倒退一步

最後直接 return result 就好



#### 範例二
arr = [-4,-1,0,2,1,3,6,-9]
target = 2

sort_arr = sorted(arr) 
sort_arr = [-9, -4, -1, 0, 1, 2, 3, 6]

result = -9 + -4 + -1 =  -14

跑陣列回圈
for first_num in range(len(sort_arr)):

設定三根指針(包含 first_num)
second_num = first_num + 1
third_num = len(sort_arr) - 1


跑 while 回圈
while second_num < third_num:

先加總三根指針目前的數字
第一圈
first_num = 0
second_num = 1
third_num = 7

total = sort_arr[first_num] + sort_arr[second_num] + sort_arr[third_num] = -9 + -4 + 6 = -7

接下來判斷，如果今天的 `用三根指針的值 - target 絕對值` < `一開始result值 - target 絕對值`
由於我們的目標是，要找尋三個值加總最接近 target 的三個數字，所以假如三個指針的數字 < 一開始的 result 值，就代表三根指針的值更接近 target
因此我們要這樣 result = total

再來我們判斷三根指針的加總值跟 target 比較
如果 < target 那就代表要把加總值變高，因此 second_num += 1
反之 > target 那就代表要把加總值變小，因此 third_num -= 1
如果相同的話，就直接回傳 result 值就好



```py
class Solution:
    def threeSumClosest(self, nums: List[int], target: int) -> int:
        sort_arr = sorted(nums)
        result = sort_arr[0] + sort_arr[1] + sort_arr[2]

        for i, first_num in enumerate(sort_arr):
            second_num, third_num = i + 1, len(sort_arr) - 1

            while second_num < third_num:
                total = first_num + sort_arr[second_num] + sort_arr[third_num]

                # 這個很重要，提前終止循環可以減少很多時間 
                if total == target:
                    return total

                if abs(total - target) < abs(result - target):
                    result = total
                
                if total < target:
                    second_num += 1
                elif total > target:
                    third_num -= 1
        
        return result        
```
