---
title: wiggle-sort
date: 2018-01-16 20:42:13
tags: [sort]
categories: sort
---
# **摇摆排序超简算法** （wiggle_sort）
------------------

+ 在github上找一些排序算法，总能发现很精辟的操作  
> from:
> [https://github.com/keon/algorithms/tree/master/sort](https://github.com/keon/algorithms/tree/master/sort)  

<!-- more -->

+  原问题好像是leetcode的题，如下：    
> Given an unsorted array nums, reorder it in-place such that  
> nums[0] <= nums[1] >= nums[2] <= nums[3]...  
> For example, given nums = [3, 5, 2, 1, 6, 4],   
>  one possible answer is [1, 6, 2, 5, 3, 4].  




----------------
+ 给出的代码(python)  
```  
    def wiggle_sort(nums):
        for i in range(len(nums)):
            if (i % 2 == 1) == (nums[i-1] > nums[i]):
                nums[i-1], nums[i] = nums[i], nums[i-1]
    if __name__ == "__main__":
        array = [3, 5, 2, 1, 6, 4] 
        print(array)
        wiggle_sort(array)
        print(array)
```

+ 代码十分简短，短到我看一眼以为是错的，然后逐行输出分析才明白  

```
    def wiggle_sort(nums):
        for i in range(len(nums)):
            if (i % 2 == 1) == (nums[i-1] > nums[i]):
                nums[i-1], nums[i] = nums[i], nums[i-1]
            print(nums)
     if __name__ == "__main__":
        array = [3, 5, 2, 1, 6, 4] 
       
        wiggle_sort(array)
       
    
```
 
+ 结果：  
```
        [3, 5, 2, 1, 6, 4] 
        [3, 5, 2, 1, 6, 4]  
        [3, 5, 2, 1, 6, 4] 
        [3, 5, 1, 2, 6, 4]
        [3, 5, 1, 6, 2, 4]
        [3, 5, 1, 6, 2, 4]
```

+ 关键在于那个if语句    
   
>  第一眼看还以为只比较偶数位与前一位(i%2)==1且nums[i-1]>nums[i]  
>  它还能比较奇数位与前一位（i%2）!=1且nums[i-1]<=nums[i]  
>  > **注意**：python中数组的-1位是最后一位，其他语言小心数组越界    

+ 等价于如下代码  
  
```
            if(i%2==1):
                if(nums[i-1]>nums[i]):
                    nums[i-1], nums[i] = nums[i], nums[i-1]
            if(i%2!=1):  
                if(nums[i-1]<=nums[i]):
                    nums[i-1], nums[i] = nums[i], nums[i-1]
```
