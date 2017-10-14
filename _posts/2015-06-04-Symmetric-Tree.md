---
layout: post
title: leetcode, 判断对称二叉树
date: 2015-06-04
categories: leetcode
---

 
 题目:

>**Symmetric Tree**

>Given a binary tree, check whether it is a mirror of itself (ie, symmetric around its center).
 
>For example, this binary tree is symmetric:
 
>```
    1
   / \
  2   2
/ \ / \
3  4 4  3
 ```
>But the following is not:
 
>```
    1
   / \
  2   2
   \   \
   3    3
``` 

>Note:
>Bonus points if you could solve it both recursively and iteratively.
 
>confused what "{1,#,2,3}" means? > read more on how binary tree is serialized on OJ.
 
>OJ's Binary Tree Serialization:
 
>The serialization of a binary tree follows a level order traversal, where '#' signifies a path terminator where no node exists below.
 
>Here's an example:
 >```
   1
  / \
 2   3
    /
   4
    \
     5
 ```

>The above binary tree is serialized as `{1,2,3,#,#,4,#,#,5}`. 

刚刚看的时候觉得挺难得, 应该, 因为你看对称啊什么的很难解决,

后来看到这个OJ的二叉树表示法的提示之后就想诶能不能通过吧二叉树转化为列表然后比较列表是否对称就行了啊

说起二叉树转化成列表就又想到了**前序, 中序, 后序**遍历

比如前序遍历, 它的遍历方法是`根->左->右`

所以如果是镜像二叉树的话, 前序遍历的结果应该是相当于原始二叉树的`根->右->左`

所以如果

原始二叉树: `根->左->右`遍历
镜像二叉树: `根->右->左`遍历

他们分别得到的列表应该是相同的. 
就用这个来判断

提交的时候发现不能完全用教科书上的遍历方法

因为有这个case:

`[1,2,2,null,3,null,3]`

就是这样的树

```
     1
   /   \
  2    2
 /  \ /  \
#   3 #   3

(# == null)
```

所以也要把空的子节点记录下来


最后A过的python代码如下

```python
# Definition for a binary tree node.
# class TreeNode:
#     def __init__(self, x):
#         self.val = x
#         self.left = None
#         self.right = None
class Solution:
    # @param {TreeNode} root
    # @return {boolean}
    def isSymmetric(self, root):
        def preorder_left_first(node):
            ret=[];
            if node == None:
                return [None]
            ret.append(node.val)
            ret = ret + preorder_left_first(node.left)
            ret = ret + preorder_left_first(node.right)
            return ret
        def preorder_right_first(node):
            ret=[];
            if node == None:
                return [None]
            ret.append(node.val)
            ret = ret + preorder_right_first(node.right)
            ret = ret + preorder_right_first(node.left)
            return ret
        # logic start here
        if root in [None,[],{}]:
            return True
        if (root.left == None and root.right == None):
            return True
        if root.left == None or root.right == None:
            return False
        left_root = root.left
        right_root = root.right
        if left_root.val != right_root.val:
            return False
        else:
            right_ret= preorder_right_first(right_root)
            left_ret = preorder_left_first(left_root)
            if right_ret == left_ret:
                return True
            else:
                return False
        

```
 
