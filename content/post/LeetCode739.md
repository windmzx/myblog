---
title: "LeetCode739: Daily Temperatures"
date: 2020-04-08T16:52:14+08:00
---

> 这是LeetCode的第739题。给定一个数组记录当前日期的温度，返回每一天的第多少天之后比当前温度高，如果没有，则返回0

# 暴力搜搜

我们首先想到的最简单的方法，就是暴力搜索。

```python
res = [0] * le
for i in range(le - 1):
    j = i + 1
    while j < le:
        if T[j] > T[i]:
            res[i] = j - i
            break
	j += 1
```

每到一个位置，我们可以向后搜索，是否存在比当前大的数，有则计算距离，无则记录0。

毫无疑问，这种算法在大数据量下会超时。其时间复杂度为$ O(n^2)$ 

# 改进1

我们注意到，不论是否存在解，我们都向后进行搜索，可以尝试对于不存在的解不进行搜索。

我们可以设置个变量n，记录一个逆向递增的序列长度，e.g. ...86,85,84,79,80...。86到79的长度为4，当我们判断res[86]的时候，我们可以判断res[80]如果为0的话，即没有比这个递增序列中的最小值79还大的数，自然就没有比86还大的数。就不需要后续搜索的过程。在这个例子中，我们发现有80，但是80不比86大，不代表后面没有比86大的数，我们还需要继续搜索。



```python
for i in range(le-2,-1,-1):
    if T[i]>=T[i+1]:
        temp+=1
        if res[i+temp]>0:
            index=i+temp
            while index <le:
                if T[index]>T[i]:
                res[i]=index-i
                break
             index+=1
            
        
        else:
            temp=0
    else:
        res[i]=1
        temp=0
```

# 改进2

不幸的是，这么改进仍然没有通过全部测试用列，主要是判断出可能有比当前位置大的时候，我们还是采用了暴力搜索的方法。我们注意到，res保存的是多少个距离之后，比当前值大。还是上面那个例子，e.g. ...86,85,84,79,80...。在86 的时候，如果res(80)大于0，即有比80大的数，我们可以不需要一个一个的进行搜索，直接跳跃到index(80)+res(80)的地方即可，多次跳跃直到终点或者res为0为止。

```python
for i in range(le-2,-1,-1):
            if T[i]>=T[i+1]:
                temp+=1
                if res[i+temp]>0:
                    index=i+temp
                    while index <le:
                        if T[index]>T[i]:
                            res[i]=index-i
                            break
                        if res[index]>0:
                            index+=res[index]
                        else:
                            break                
                else:
                    temp=0
            else:
                res[i]=1
                temp=0
        return res
```



这个改进额能通过所有的测试用例。