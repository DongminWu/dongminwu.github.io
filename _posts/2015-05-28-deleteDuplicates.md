---
layout: post
title: leetcode, deleteDuplicates删除重复链表
date: 2015-05-28
categories: leetcode
---

问题

>Remove Duplicates from Sorted List 

>Given a sorted linked list, delete all duplicates such that each element appear only once.

>For example,

>Given 1->1->2, return 1->2.

>Given 1->1->2->3->3, return 1->2->3. 


这题终于是自己搞定的了

一开始的时候是觉得这就是个简单的链表所以一次遍历把所有的重复的链表接上就好了

```python
# Definition for singly-linked list.
# class ListNode:
#     def __init__(self, x):
#         self.val = x
#         self.next = None

class Solution:
    # @param {ListNode} head
    # @return {ListNode}
    def deleteDuplicates(self, head):
        if head == None or head == [] or head == {}:
            return None
        tmp=head
        while (tmp.next != None):
            if tmp.val == tmp.next.val:
                tmp.next = tmp.next.next
            if tmp.next != None:
                tmp=tmp.next
        return head
```


后来发现在[1,1,1,1,1]
这种重复数据的case里面会出错

于是重复调用了自己, 然后这样就能消除多个重复值的影响, 

```python
# Definition for singly-linked list.
# class ListNode:
#     def __init__(self, x):
#         self.val = x
#         self.next = None

class Solution:
    # @param {ListNode} head
    # @return {ListNode}
    def deleteDuplicates(self, head):
        if head == None or head == [] or head == {}:
            return None
        tmp=head
        while (tmp.next != None):
            if tmp.val == tmp.next.val:
                tmp.next = tmp.next.next
                self.deleteDuplicates(tmp)
            if tmp.next != None:
                tmp=tmp.next
        return head
```

但是这样的时间效率好低啊...

时间复杂度应该是O(n2), 如果没算错的话

看看别人的解法


```java
public class Solution {
    public ListNode deleteDuplicates(ListNode head) {
        if(head == null || head.next == null)
            return head;
 
        ListNode prev = head;    
        ListNode p = head.next;
 
        while(p != null){
            if(p.val == prev.val){
                prev.next = p.next;
                p = p.next;
                //no change prev
            }else{
                prev = p;
                p = p.next; 
            }
        }
 
        return head;
    }
}
```


老子写的真垃圾...

直接通过判断是不是一样的决定指针走不走不就行了...

```python
# Definition for singly-linked list.
# class ListNode:
#     def __init__(self, x):
#         self.val = x
#         self.next = None

class Solution:
    # @param {ListNode} head
    # @return {ListNode}
    def deleteDuplicates(self, head):
        if head == None or head == [] or head == {}:
            return None
        tmp=head
        while (tmp.next != None):
            if tmp.val == tmp.next.val:
                tmp.next = tmp.next.next
            else:
                tmp=tmp.next
        return head

```

运行时间一下从快700ms降到了80多ms

!!!!


