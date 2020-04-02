---
title: "搜索旋转排序数组"
date: 2020-04-02T16:56:00+08:00
categories:
- 算法
---

这个题要求我们在一个经过旋转的排序数组中进行搜索。而且要求二分查找进行搜索。题目难以理解，其实就是二分的时候，先判断有序的那一半。因为有序我们就可以通过边界判断我们的目标值是否在有序的这一半之中。
这个数组每次分割都可以分为旋转和不旋转的两部分。依次迭代。
```python3
from typing import List
class Solution:
    def search(self, nums: List[int], target: int) -> int:
        left=0
        right=len(nums)-1
        while left<=right:
            mid=(left+right)//2
            if nums[mid]==target:
                return mid

            if nums[mid]<nums[right]:
                # 右边是有序的
                if nums[mid]<target and nums[right]>=target:
                    left=mid+1
                else:
                    right=mid-1
            else:
                # 左边是有序的
                if nums[mid]>target and nums[left]<=target:
                    right=mid-1
                else:
                    left=mid+1
        return -1
if __name__ == "__main__":
    x=Solution()
    print(x.search([4,5,6,1,2,3],1))
```


其中要注意的点：
- 左半部分的边界要小于可能等于target
- 右半部分的边界要大于等于target
- left和right可以重合，如果只是小于的话，或找不到答案。