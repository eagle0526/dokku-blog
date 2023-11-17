---
title: 179. largest Number
sidebar_label: "179. largest Number"
description: LeetCode
last_update:
  date: 2023-11-16
keywords:
  - LeetCode  
sidebar_position: 1
---

### 題目

Given a list of non-negative integers nums, arrange them such that they form the largest number and return it.   
Since the result may be very large, so you need to return a string instead of an integer.  

```md
Example 1:

Input: nums = [10,2]
Output: "210"
Example 2:

Input: nums = [3,30,34,5,9]
Output: "9534330"
```

### 題目翻譯

這一題的目的是，題目給你一個陣列，裡面有很多的正整數，我們要把裡面陣列重新排列，排出一個最大的數字，並且最後要返回一個字串

### python 解題

```py
from functools import cmp_to_key

class Solution(object):
    def largestNumber(self, nums):
        """
        :type nums: List[int]
        :rtype: str
        """
        
        def compare(x, y):
            return int(x+y) - int(y+x)
        sorted_nums = sorted(map(str, nums), key=cmp_to_key(compare), reverse=True)

        result = ''.join(sorted_nums)
        return str(int(result))
```

解題詳解:
1. 先引入 `cmp_to_key` 這個函式，這個函式主要是用來排列一個陣列裡面的數值
2. 新增一個 `compare` 函式，主要是來比較兩個數值的大小， -> ex. 假設今天 x=2, y=5 並且兩個數值是字串，就會像這樣 -> (25-52)，返回 -27
3. `sorted` 可以排列一個陣列裡面的數值，首先用 `map` 把陣列所有的值轉成 `str`，轉成 `str` 後用 cmp_to_key(compare) 來處理這個陣列，最後可以得到一個整理過後的陣列
4. `reverse=True` 因為我們要降序排列，最大的值在前面，所以要降序
5. 最後把陣列的值用 `join` 串起來，就完成了


### 詳細解釋 cmp_to_key(compare)

我們實際把 `sorted_nums` 印出來看一下，這邊在做什麼事情，首先看一下這段程式：
```py
from functools import cmp_to_key
arr = [1, 30, 5, 3]

def compare_arr(x, y):
    print("x:" ,x ,"y: ", y )
    print("x+y:" ,x+y ,"y+x: ", y+x )
    print("**")
    return int(x+y) - int(y+x) 

print(sorted(map(str, arr), key=cmp_to_key(compare_arr), reverse=True))
```

來看一下印出來的樣子：
```md
x: 5 y:  3
x+y: 53 y+x:  35
**
x: 30 y:  5
x+y: 305 y+x:  530
**
x: 30 y:  5
x+y: 305 y+x:  530
**
x: 30 y:  3
x+y: 303 y+x:  330
**
x: 1 y:  3
x+y: 13 y+x:  31
**
x: 1 y:  30
x+y: 130 y+x:  301
**
['5', '3', '30', '1']
```

這樣就可以發現，我們在使用 sorted 函數時，我們先用 map 函式把陣列裡面的數字都轉成 `str`，後面的 `cmp_to_key(compare_arr)` 接到這些 `str` 後，會進行迭代比較。  
如果 int(x + y) 的結果大於 int(y + x)，那麼 x 在排序中應該排在 y 的前面；反之，如果 int(x + y) 小於 int(y + x)，則 x 應該排在 y 的後面。這樣的比較保證了拼接後的數字最大。   
以上面案例來說， `x: 5, y:  3 , x+y: 53 y+x:  35, 因為 x+y > y+x, 因此 x 會在 y 的前面`。







