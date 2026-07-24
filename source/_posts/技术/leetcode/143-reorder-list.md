---
title: LeetCode 143. 重排链表
date: 2024-01-15T19:32:06+08:00
updated:
tags: [LeetCode, 链表, 双指针]
categories: [技术, LeetCode]
keywords: []
description: LeetCode 143. 重排链表
top_img:
comments:
cover:
toc:
---
## 题目描述

给定一个单链表 `L` 的头节点 `head` ，单链表 `L` 表示为：

> L0 → L1 → … → Ln - 1 → Ln

请将其重新排列后变为：

> L0 → Ln → L1 → Ln - 1 → L2 → Ln - 2 → …

不能只是单纯的改变节点内部的值，而是需要实际的进行节点交换。
![Pasted image 20240115180529 1](https://raw.githubusercontent.com/LosLiSang/picgo/main/Pasted%20image%2020240115180529%201.png)

## 题解

### 解法一：暴力解

第i次循环都把最后一个节点移动到第`2*i-1`个节点后面，当然不是用for循环，只是代码运行的时候链表内部是这样的过程。

记itr为从前往后遍历的节点，只要itr.next不为null，就一直做以下循环

1. 找到最后一个节点的前一个节点记为pre，最后一个节点记为last = pre.next
2. 把最后一个节点删除，插入到第itr的后面
3. 令itr前移动两个单位itr = itr.next.next

步骤2的Code

```java
pre.next = last.next;
last.next = itr.next;
itr.next = last
```

#### 代码

```java
class Solution {
    public ListNode lastPre(ListNode head){
        ListNode itr = head;
        while (itr.next.next != null) {
            itr = itr.next;
        }
        return itr;
    }
    
    public void reorderList(ListNode head) {
        ListNode itr = head;
        while(itr != null && itr.next != null){
            ListNode pre = lastPre(head);
            ListNode last = pre.next;
            pre.next = last.next;
            last.next = itr.next;
            itr.next = last;
            itr = itr.next.next;
        }
    }
}
```

时间复杂度 $O(N^2)$

### 解法二

1. 找中点，拆分链表
2. 倒置后半段链表
3. 合并两段链表

#### 找中点（快慢指针法）

##### eg1

1->2->3->4

fast:1    slow:1

fast:3    slow:2

fast.next.next为null，终止循环

那么中点就记为mid = slow.next 为 3

##### eg2

1->2->3->4->5

fast:1    slow:1

fast:3    slow:2

fast:5    slow:3

fast.next为null，终止循环

那么中点就记为mid = slow.next 为 4

拆分链表，slow.next = null;

```java
public ListNode getMid(ListNode head){
    ListNode slow = head, fast = head;
    while(fast != null && fast.next != null){
        slow = slow.next;
        fast = fast.next.next;
    }
    return slow;
}
```

#### 倒置链表

记后半段链表为back，前半段链表为front

倒置链表使用前插法

```java
public ListNode reverse(ListNode head){
     ListNode itr = head;
     ListNode dummyHead = new ListNode();
     while(itr != null){
         ListNode temp = itr.next;
         itr.next = dummyHead.next;
         dummyHead.next = itr;
         itr = temp;
    }
    return dummyHead.next;
}
```

#### 合并链表

```java
public ListNode merge (ListNode l1, ListNode l2) {
    ListNode head = l1;
    while(l2 != null && l1 != null){
        ListNode l1Next = l1.next;
        l1.next = l2;   
        l2 = l2.next;
        l1.next.next = l1Next;
        l1 = l1.next.next;
    }
    return head;
}
```

## 总结

1. 链表的赋值，其实和变量交换很类似，都是需要备份的。
2. 封装好功能的函数，远远比一整段写出来的代码要更好理解！！
3. 不用害怕定义变量，但是前提是你要知道这个变量是用来做什么的
