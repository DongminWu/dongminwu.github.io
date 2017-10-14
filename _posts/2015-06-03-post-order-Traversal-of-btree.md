---
layout: post
title: leetcode, 非递归, 实现后序遍历二叉树
date: 2015-06-03
categories: leetcode
---

题目:

>**Binary Tree Postorder Traversal** 

>Given a binary tree, return the postorder traversal of its nodes' values.

>For example:
>Given binary tree {1,#,2,3},

>  1
>    \
>    2
>    /
>   3

>return [3,2,1].

>Note: Recursive solution is trivial, could you do it iteratively?


这个问题貌似是一个挺经典的面试题

先要搞清楚后序遍历是什么

>先遍历左节点, 然后右节点, 最后根节点

百度百科里面的递归算法

```cpp
struct btnode
{
    int d;
    struct btnode *lchild;
    struct btnode *rchild;
};
void postrav(struct btnode *bt)
{
    if(bt!=NULL)
    {
        postrav(bt->lchild);
        postrav(bt->rchild);
        printf("%d ",bt->d);
    }
}
```


思路是这样的. 虽然有点复杂

有两个堆栈, 一个用来存每个叶子的节点的地址 叫做: `pointer_stack`,

 另一个用来存这个节点是否被访问过,叫做 `flag_stack`

其实两个堆栈的指针都是走的同一个位置...

把节点存入`pointer_stack`堆栈的时候, 如果这个节点被访问过的话, `flag_stack`堆栈会被相应的存入 一个 1

否则存入 0

如果没有子节点可以遍历了就弹出堆栈里面的节点继续遍历.

恩, 于是A过的代码是:


```python
# Definition for a binary tree node.
# class TreeNode:
#     def __init__(self, x):
#         self.val = x
#         self.left = None
#         self.right = None
class Stack:
    def __init__(self):
        self.top = -1
        self.stack=[];
    def pop(self):
        if self.top  < 0:
            return False
        else: 
            ret = self.stack[self.top]
            del self.stack[self.top]
            self.top = self.top - 1
            return ret
    def push(self, x):
        self.top = self.top + 1
        self.stack.append(x)
        return True
    def isEmpty(self):
        if self.top < 0:
            return True
        else:
            return False
    def peek(self):
        if self.top < 0:
            return None
        else:
            return self.stack[self.top]
    def dump(self):
        print self.stack , "top:", self.top
    def bottomValue(self):
        if self.top < 0:
            return None
        else:
            return self.stack[0]
class Solution:
    # @param {TreeNode} root
    # @return {integer[]}
    def postorderTraversal(self, root):
        pointer_stack = Stack();
        flag_stack = Stack();
        ret_list=[]
        if root == None:
            return []
        pointer_stack.push(root)
        flag_stack.push(0)
        while (not pointer_stack.isEmpty()):
            pointer = pointer_stack.pop()
            flag = flag_stack.pop()
            if (flag == 0):
                if pointer.left != None:
                    #1. if has left child, push current position first
                    flag_stack.push(1) #has been visited
                    pointer_stack.push(pointer)
                    #1.2. if right child != None, push right child
                    if pointer.right != None:
                        flag_stack.push(0)
                        pointer_stack.push(pointer.right)
                    #1.3. push left child, due to that, left child will be visited firstly
                    flag_stack.push(0) 
                    pointer_stack.push(pointer.left)
                    continue;
                if pointer.right != None:
                    # 2. if no left and has right, only push current position
                    flag_stack.push(1) #has been visited
                    pointer_stack.push(pointer)
                    # 2.1 push right child
                    flag_stack.push(0) 
                    pointer_stack.push(pointer.right)
                    continue;
                if (pointer.left == None and pointer.right == None):
                    ret_list.append(pointer.val);
            elif (flag == 1):
                ret_list.append(pointer.val)
        return ret_list
```


看看百度百科里面的非递归算法

好像和我的差不多. 只不过它的二叉树的节点里面可以存储一个tag, 标记有没有被访问过


再[在网上看到了另外一个不用堆栈的方法, 就是非递归,非堆栈](http://blog.chinaunix.net/uid-7897183-id-75493.html), 

也是巧妙的构造了一个特殊的可以存储父节点的二叉树节点来做的

引用:

```cpp

/**************************************************************
 * $ID: PostOrderWalkOfBST.h
 * $DESC: postorder walk a BST, without stack, no recursion
 * $AUTHOR: rockins chen
 * $DATE: Mon Sep 24 23:15:55 CST 2007
 * $BUG: ybc2084@163.com or
 *        www.dormforce.net/blog/rockins
 **************************************************************/
 
#ifndef    _POST_ORDER_WALK_OF_BST_H_
#define    _POST_ORDER_WALK_OF_BST_H_
//
// BST node definition
//
typedef struct _BST_NODE {
    void    * private_data;        // store private data
    int        key;
    struct _BST_NODE * parent;
    struct _BST_NODE * lchild;
    struct _BST_NODE * rchild;
} BST_NODE, *PBST_NODE;
//
// post order walk a BST
// root denotes the BST
//
extern int PostOrderWalkBST(PBST_NODE root);
//
// find first postorder node of BST
// root denotes the BST(maybe a subtree)
//
static PBST_NODE FindFirstPostOrderNodeOfBST(PBST_NODE root);
#endif
/**************************************************************
 * $ID: PostOrderWalkOfBST.c
 * $DESC: postorder walk a bst, without stack, no recursion
 * $AUTHOR: rockins chen
 * $DATE: Mon Sep 24 23:17:03 CST 2007
 * $BUG: ybc2084@163.com or
 *        www.dormforce.net/blog/rockins
 **************************************************************/
#include "PostOrderWalkOfBST.h"
int
PostOrderWalkBST(PBST_NODE root)
{
    PBST_NODE curr_node;
    PBST_NODE next_node;
    
    // fist go to the first postorder node of BST
    curr_node = FindFirstPostOrderNodeOfBST(root);
    
    // if current node's parent is NULL, then curr_node must be root of BST, finished
    while (!curr_node->parent) {
    
        // if current node is parent node's left child and parent has no right child,
        // then parent is next node to visit
        if (curr_node->parent->lchild == curr_node &&
            !curr_node->parent->rchild)
            next_node = curr_node->parent;
            
        // if current node is parent node's left child and parent has right child,
        // then the fist postorder node of parent's right subtree is next node to visit
        if (curr_node->parent->lchild == curr_node &&
            curr_node->parent->rchild)
            next_node = FindFirstPostOrderNodeOfBST(
                            curr_node->parent->rchild);
        
        // if current node is parent node's right child(implicitly indicate parent node is not null),
        // then parent node is just next node
        if (curr_node->parent->rchild == curr_node)
            next_node = curr_node->parent;
        
        //
        // XXX: now visit curr_node, if necessary
        //
        VisitNode(curr_node);
        
        // step
        curr_node = next_node;
    }
}
//
// find fist postorder node of BST
//
static PBST_NODE
FindFirstPostOrderNodeOfBST(PBST_NODE root)
{
    PBST_NODE pNode;
    
    pNode = root;
    
    while (pNode->lchild || pNode->rchild) {
        if (pNode->lchild)
            pNode = pNode->lchild;
        else if (pNode->rchild)
            pNode = pNode->rchild;
    }
    
    return (pNode);
}
//
// virtual visiting pNode, do nothing
//
static int
VisitNode(PBST_NODE pNode)
{
}


```
