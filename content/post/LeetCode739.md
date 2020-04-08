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