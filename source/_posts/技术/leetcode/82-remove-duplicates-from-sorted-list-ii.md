---
title: LeetCode 82. 删除排序链表中的重复元素 II
date: 2024-01-15T19:40:04+08:00
updated:
tags: [LeetCode, 链表, 双指针]
categories: [技术, LeetCode]
keywords: []
description: 82. 删除排序链表中的重复元素 II
top_img:
comments:
cover:
toc:
---
[82. 删除排序链表中的重复元素 II](https://leetcode.cn/problems/remove-duplicates-from-sorted-list-ii/)

## 题目描述

给定一个已排序的链表的头 `head` ， _删除原始链表中所有重复数字的节点，只留下不同的数字_ 。返回 _已排序的链表_ 。

### 示例

![Pasted image 20240115145926](https://raw.githubusercontent.com/LosLiSang/picgo/main/Pasted%20image%2020240115145926.png)

## 题解

首先，链表是排序过的，所以相同的节点肯定是连续的，只需要遍历一遍就能把所有相同的节点删除

记pre为去重链表的遍历节点itr的前一个节点。（我们从前往后遍历，前面的链表就是去除过后的链表）

1. 如果itr.val == itr.next.val，那我们就记flag为当前遍历节点的值，令`flag=itr.val`，不断向后遍历令itr = itr.next，直到，itr.next.val != flag 或者 itr.next == null。然后让pre.next = itr.next
2. 如果itr.val != itr.next.val，那么就说明itr不是冗余节点，则不必删除，让pre = itr即可

### 代码

```java
class Solution{
    public ListNode deleteDuplicates(ListNode head){
        int flag = -101;
        ListNode dummyNode = new ListNode();
        dummyNode.next = head;
        ListNode pre = dummyNode;
        ListNode itr = pre.next;
        while(itr != null && itr.next != null){
            if(itr.val == itr.next.val){
                flag = itr.val;
                while(itr.next != null && itr.next.val == flag){
                    itr = itr.next;
                }
                pre.next = itr.next;
                itr = pre.next;
            }else{
                pre = itr;
                itr = itr.next;
            }   
        }
        return dummyNode.next;
    }
}
```

时间复杂度 O(n), 空间复杂度O(1)

## 总结

链表的题通常需要注意两点：

1. 舍得用变量，千万别想着节省变量，否则容易被逻辑绕晕
2. head 有可能需要改动时，先增加一个 假head，返回的时候直接取 假head.next，这样就不需要为修改 head 增加一大堆逻辑了。
