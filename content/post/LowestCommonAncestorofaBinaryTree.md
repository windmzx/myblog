---
title: "二叉树最近的公共祖先节点"
date: 2020-04-03T14:32:07+08:00
categories:
- 算法
---

> 给定一个二叉树, 找到该树中两个指定节点的最近公共祖先。
>
> 百度百科中最近公共祖先的定义为：“对于有根树 T 的两个结点 p、q，最近公共祖先表示为一个结点 x，满足 x 是 p、q 的祖先且 x 的深度尽可能大（一个节点也可以是它自己的祖先）。”
>
> 来源：力扣（LeetCode）
> 链接：https://leetcode-cn.com/problems/lowest-common-ancestor-of-a-binary-tree
> 著作权归领扣网络所有。商业转载请联系官方授权，非商业转载请注明出处。

这个题，我一开始陷入了误区，我想到的事，分别递归的便利左右子树，如果字数中包含p或者q节点，我们就返回True，如果一直到None都没有找到p或者q，说明这一子树中没有不可能是两个节点的最近公告祖先。如果一个节点的左右子树返回的都是True，那么这个节点就是祖先节点。

但是这样的算法笔记比较麻烦。需要写helper函数进行递归，而helper的返回值是布尔值不能直接返回给题目的要求。

查看评论区，我们可以发现一个新思路，用Null来表示这一支没有q或者p，因为当我们遍历到子树的叶节点还没有p、q，返回的就是null，如果有，返回的则是q或者q的指针。与布尔值进行判断相同。如果我们返回的两个指针都是有值的，说明当前节点的左右子树中包含p、q且是最近的。

这样的算法就很简洁了。

```python
# Definition for a binary tree node.
# class TreeNode:
#     def __init__(self, x):
#         self.val = x
#         self.left = None
#         self.right = None


class Solution:
    def lowestCommonAncestor(self, root: 'TreeNode', p: 'TreeNode', q: 'TreeNode') -> 'TreeNode':
        if root is None or root == q or root == p:
           return root
        r1=self.lowestCommonAncestor(root.left,p,q)
        r2=self.lowestCommonAncestor(root.right,p,q)
        if r1 and r2:
            return root 
        elif r1:
            return r1
        else:
            return r2

```

