---
title: "下一个组合"
date: 2020-04-02T18:31:40+08:00
draft: true
---
> 实现获取下一个排列的函数，算法需要将给定数字序列重新排列成字典序中下一个更大的排列。
如果不存在下一个更大的排列，则将数字重新排列成最小的排列（即升序排列）。

>必须原地修改，只允许使用额外常数空间。

>以下是一些例子，输入位于左侧列，其相应输出位于右侧列。
1,2,3 → 1,3,2
3,2,1 → 1,2,3
1,1,5 → 1,5,1


```python
from typing import List
class Solution:
    def nextPermutation(self, nums: List[int]) -> None:
        def swap(i,j):
            temp=nums[i]
            nums[i]=nums[j]
            nums[j]=temp
        """
        Do not return anything, modify nums in-place instead.
        """
        if len(nums)==1:
            return
        i=len(nums)-1
        while(i-1>=0 and nums[i]<=nums[i-1]):
            i-=1
        # i就是要交换的位置
        if i!=0:
            j=len(nums)-1
            while j>=0 and nums[j]<=nums[i-1] :
                j-=1
            swap(i-1,j)
        k=len(nums)-1

        while i<k:
            swap(i,k)
            i+=1
            k-=1

        print(nums)
        
if __name__ == "__main__":
    x=Solution()
    x.nextPermutation([5,1,1])

```
这个题的目标为找出给定排列的下一个字典序。字典序是递增的，因为我们要找到给定序列中的字典序的部分（a,b）将a换成后面数字中仅大于a的。然后将b开始的部分进行重新排序就行。 由于后面本身就是字典序，因此我们只需要两两交换顺序就可以。