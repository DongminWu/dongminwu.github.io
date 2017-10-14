---
layout: post
title:leetcode, 通过先序遍历和中序遍历构造二叉树
date: 2015-06-07
categories: leetcode
---


题目: 


>**Construct Binary Tree from Preorder and Inorder Traversal**

>Given preorder and inorder traversal of a tree, construct the binary tree.

>Note:
>You may assume that duplicates do not exist in the tree. 





首先是这个二叉树里面的值没有重复的才行.. 



否则就怎么也解不出来的..



想了个思路:



比如有这样一个二叉树



```

    1

   / \

  2   3

 / \   \

4   5   9

   / \

  6   7

       \

        8

```

他的先序遍历是:  `preorder = [1, 2, 4, 5, 6, 7, 8, 3, 9]`

他的中序遍历是:  `in_order = [4, 2, 6, 5, 7, 8, 1, 3, 9]`





能显而易见看出的是先序遍历的第一个元素永远是根节点..



但是根节点之后的一个数可能是左节点也可能是右节点



而中序遍历的列表里面 根节点`1` 左边(`4, 2, 6, 5, 7, 8`)的都是左子树的元素, 右边(`3, 9`)的都是右子树元素



所以我有了这样的假设





>**if** 在in_order中的根节点位置的左边有数据 **then** preorder[1]是左子树的根节点

>**if** 在in_order中的根节点位置的右边有数据 **then** preorder[1]是右子树的根节点





扩展一下



就变成了

>**if** 在in_order中某个位置的左边有数据 **then** preorder[对应节点的索引+1]是左子树的根节点

>**if** 在in_order中某个位置的右边有数据 **then** preorder[对应节点的索引+1]是右子树的根节点





反过来想, 



以preorder为条件的话.



那么可以变为这个流程



1. 假设`preorder[index] == data`

2. 如果`preorder[]` 有未被读取过的数据(可能是这种情况 [ 读过, **读过**, 读过, 未读过]<- 指针在第2个**读过**那里)

3. 那么找到 `in_order[in_index] == data`

4. 如果`preorder[]` 的数据在 `in_order[in_index]` 左边

5. 那么`preorder[]` 是`preorder[index]`的左节点

6. 如果在右边就是右节点啦



于是就可以递归上面的步骤, 直到in_order中的某个位置的数据两边都没有未被访问过数据, 也就是说是叶子节点



卧槽搞了3-4个小时..就为了实现这个逻辑..





终于尼玛A过了,,,看起来简单的算法实现起来真尼玛麻烦



主要问题出在 `hasZero()` 这个函数上面, 之前的实现是


```python

 def hasZero(direct,search_list, index):

            if direct == "left":

                try:

                    search_list[0:index].index(0)

                except ValueError:

                    return False

                return True

            if direct == "right":

                try:

                    search_list[index:(len(search_list)-1)].index(0)

                except ValueError:

                    return False

                return True


```

这样的话只要`index`的左侧或者右侧有0

这个函数的返回值就是`True`





不过我觉得我的实现好垃圾..



因为运行时间竟然到了916ms, 平均水平我觉得应该是200ms左右...








我的代码

```python

def buildTree(self, preorder, inorder):
        def hasZero(direct,search_list, index):
            pointer= index
            if direct == "left":
                if index <= 0:
                    # print "left False"
                    return False
                pointer -= 1
                while (pointer >= 0):
                    if (search_list[pointer] == 1 and pointer < index -1):
                        # print "left True"
                        return True
                    elif (search_list[pointer] == 1 and pointer >= index -1):
                        # print "left False"
                        return False
                    pointer -= 1
                else:
                    # print "left True"
                    return True
            if direct == "right":
                if index >= len(search_list)-1:
                    # print "right False"
                    return False
                pointer += 1
                while (pointer <= len(search_list)-1):
                    # print pointer
                    if (search_list[pointer] == 1 and pointer > index +1):
                        # print "right True"
                        return True
                    if (search_list[pointer] == 1 and pointer <= index +1):
                        # print "right False"
                        return False
                    pointer += 1
                else:
                    # print "right True"
                    return True
                    
        def recursive(pre_visited, in_visited, preorder, inorder, current_node, pre_index):
            in_index = inorder.index(preorder[pre_index])
        #    if pre_index >= len(preorder)+1:
                #return None
            pre_visited[pre_index] = 1
            in_visited[in_index] = 1
            current_node.val = preorder[pre_index] 
            # print "recursive[", pre_index,"]:in_visited", in_visited, ":pre_visited:" ,pre_visited
            if hasZero("left", in_visited, in_index):
                #there are some un-visited nodes left
                try:
                    next_index = pre_visited.index(0)
                except ValueError:
                    assert(True)
                else:
                    # print "[",pre_index,"] -> left"
                    current_node.left = TreeNode(None)
                    recursive(pre_visited,in_visited,preorder,inorder,current_node.left,next_index)
            if hasZero("right", in_visited, in_index):
                #there are some un-visited nodes right
                try:
                    next_index = pre_visited.index(0)
                except ValueError:
                    assert(True)
                else:
                    # print "[",pre_index,"] -> right"
                    current_node.right = TreeNode(None)
                    recursive(pre_visited,in_visited,preorder,inorder,current_node.right,next_index)
            # print "leaf.val", current_node.val
            return
        if inorder in  [None, [], {}]:
            return None
        if preorder in  [None, [], {}]:
            return None
        root_node = TreeNode(None)
        pre_visited = [0]*len(preorder)
        in_visited = [0]*len(inorder)
        recursive(pre_visited,in_visited,preorder,inorder,root_node,0)
        return root_node
        
```



