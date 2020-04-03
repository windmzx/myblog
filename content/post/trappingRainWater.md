---
title: "LeetCode(42)TrappingRainWater"
date: 2020-04-03T09:52:27+08:00
categories:
- 算法
---

> 接雨水，给定一个高度数组，求出这个高度数组构成的图形能接住多少雨水。

装水的问题，我们首先想到的就是最短木桶原理。对于每一个位置，它能装的水的容量取决于 左右两边的目前最高的木板 中较短的那一根。



因为我们要用最短的木板，因此我们从两边开始遍历，设置两个标志位分别记录坐标的最高值和右边的最高值。每次更新较短的左边或者右边。

每次更新的时候，如果当前位置比最高值低，则说明这个位置能装水，更新容量。如果比最高值高，则更新相应的最高值。

```python
from typing import List
class Solution:
    def trap(self, height: List[int]) -> int:
        if len(height)<=2:
            return 0
        left,right=1,len(height)-2
        leftmax=height[left-1]
        rightmax=height[right+1]
        res=0
        while left<=right:
            if leftmax<rightmax:
                if height[left]>leftmax:
                    leftmax=height[left]
                else:
                    res+=leftmax-height[left]
                left+=1
            else:
                if height[right]>rightmax:
                    rightmax=height[right]
                else:
                    res+=rightmax-height[right]
                right-=1
        return res
if __name__ == "__main__":
    x=Solution()
    print(x.trap([1,0,2,0,3]))
                
```

