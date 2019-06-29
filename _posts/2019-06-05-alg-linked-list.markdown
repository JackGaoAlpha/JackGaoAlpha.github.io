---
layout: post
title: 数据结构和算法（01）：链表
date: 2019-06-05 01:01:01.000000000 +09:00
mermaid: true
---

## 链表结构

![](http://ww4.sinaimg.cn/large/006tNc79ly1g4hslsz71zj30h4061mxb.jpg)

![](http://ww3.sinaimg.cn/large/006tNc79ly1g4hsm586h2j30h5062wen.jpg)

![](http://ww4.sinaimg.cn/large/006tNc79ly1g4hsmejoe8j30h6065weo.jpg)

![](http://ww2.sinaimg.cn/large/006tNc79ly1g4hsml8sv9j30gw07kq3a.jpg)


## 时间复杂度以及双向链表的优势

理论上，单链表的插入、删除操作时间复杂度都是 O(1) ，而随机访问元素的时间复杂度是 O(n)。

在实际的软件开发中，从链表中删除一个数据无外乎这两种情况：

1. 删除结点中“值等于某个给定值”的结点；
2. 删除给定指针指向的结点。

对于第一种情况，无论是单链表还是双向链表，单纯删除操作时间复杂度是 O(1)，但是遍历查找复杂度是 O(n)。

对于第二种情况，双向链表可以直接获取前驱节点再执行操作，时间复杂度是 O(1)。

对于一个有序链表，双向链表的按值查询的效率比较高 —— 因为可以记录上次查找的位置 p，每次查询时，根据要查找的值与 p 的大小关系，决定是往前还是往后查找，所以平均只需要查找一半的数据。


## 和数组的对比

* 数组

1. 连续内存空间，可以借助 CPU 缓存机制，访问效率高。
2. 大小固定，扩容时数组拷贝费时

* 链表

1. 不是连续存储，无法有效预读。
2. 天然支持动态扩容

## 相关算法题总结

### LeetCode 876. Middle of the Linked List

寻找单向链表的中间节点。

可以设置两个指针 fast 和 slow 指向 head。以 fast 步长为 2，slow 步长为 1开始循环遍历链表。当 fast 指针遍历到尾节点或者 NULL 时结束循环。此时，slow 指向的节点就是中间节点。

### LeetCode 206. Reverse Linked List

反转单向链表。

设链表为 head -> ... -> cur -> a -> b。则获取 a 之后，cur.next = b，a.next= head，head = a。完成一个反转。循环直至结束。

核心代码如下。

```python
cur = head
while cur is not None:
    if cur.next is not None:
        n = cur.next
        cur.next = n.next
        n.next = head
        head = n
    else:
        break
return head
```

### LeetCode 237. Delete Node in a Linked List

给定 node 指针删除单向链表节点。

直接使用 node.next.val 替换 node.val，然后改变指针指向。

### LeetCode 21. Merge Two Sorted Lists

合并两个排序链表。

设置冗余节点 dummy 作为头结点。双指针遍历链表接到 dummy 之后，返回 dummy.next。

### LeetCode 83. Remove Duplicates from Sorted List

移除排序链表的重复元素。

```python

cur = head
while cur and cur.next:
    if cur.next.val == cur.val:
        cur.next = cur.next.next
    else:
        cur = cur.next
return head
```

### LeetCode 141. Linked List Cycle

检查链表是否循环链表。

设置快慢指针 fast （步长为 2） 和 slow （步长为 1）。如果有一个遍历到空值，则不是循环链表。如果有有一个追上另一个（表现为 fast.val 等于 slow.val），则存在循环。

### LeetCode 234. Palindrome Linked List

检查单向链表是否回文链表。

首先找到链表的中间节点（见 LeetCode 876）。此时 slow 为中间节点。

然后反转 slow 之后的节点（见 LeetCode 206）。

此时，如果是回文链表则可以从头指针和中间节点指针后遍历得到相同的值。

```python
class Solution(object):
    def isPalindrome(self, head):
        """
        :type head: ListNode
        :rtype: bool
        """
        slow = fast = head
        while fast and fast.next:
            fast = fast.next.next
            slow = slow.next
        node = None
        while slow:
            nextNode = slow.next
            slow.next = node
            node = slow
            slow = nextNode
        while node:
            if node.val != head.val:
                return False
            node = node.next
            head = head.next
        return True
```

### LeetCode 203. Remove Linked List Elements

移除链表中值为 val 的所有元素。

设置 dummy 指针指向 head 节点。设置 cur = dummy。遍历链表，如 cur.next 等于 val，则 cur.next = cur.next.next。返回 dummy.next。

### LeetCode 160. Intersection of Two Linked Lists

获取两个单项链表的交点（如下图，返回 8），不相交则返回 NULL。

![](http://ww4.sinaimg.cn/large/006tNc79ly1g4hsmvvf4oj30cd03umx1.jpg)

设置指针 pa 和 pb 指向各自链表的访问位置，开始遍历链表。

当遍历到 null 节点时指针指向另一个链表。如果这个过程中遍历到值相等的节点，则返回。

> 设 len(A) = a + c，a 为相交节点及之前节点长度，c 为之后。len(B) = b + c。
> 如果存在相交节点，则有 (a + c) + b == (b + c) + a

### LeetCode 328. Odd Even Linked List

将单向链表中，索引计数为双数的节点，接到索引计数为单数的节点后。如：

```
Input: 1->2->3->4->5->NULL
Output: 1->3->5->2->4->NULL
```

代码如下：

```python
class Solution:
    def oddEvenList(self, head: ListNode) -> ListNode:
        if head is None:
            return head
        odd_head = odd = head
        even_head = even = head.next
        while odd and even:
            if even.next:
                odd.next = even.next
                odd = odd.next
                even.next = odd.next
                even = even.next
            else:
                break
        odd.next = even_head  # 记得把 odd 指针指向 even 链表部分的 “头” 节点
        return head
```

### LeetCode 86. Partition List

给定链表和一个值 x，把链表中小于 x 的节点全部放到大于等于 x 的节点前，这两部分（小于 和 大于等于）需要保留各自的原始顺序。比如：

```
Input: head = 1->4->3->2->5->2, x = 3
Output: 1->2->2->4->3->5
```

设置两个链表partition 各自的 dummy 指针和当前指针。遍历链表，如果当前的节点小于 x ，则接到 小于的那部分，反之为大于等于那部分，更新当前指针和遍历链表的指针。代码如下：

```python
class Solution:
    def partition(self, head: ListNode, x: int) -> ListNode:
        dummy_lt = lt = ListNode(0)
        dummy_gt = gt = ListNode(0)
        while head:
            if head.val < x:
                lt.next = head
                lt = lt.next
            else:
                gt.next = head
                gt = gt.next
            head = head.next
        gt.next = None
        lt.next = dummy_gt.next
        return dummy_lt.next
```

### LeetCode 148. Sort List

对链表进行排序，要求 O(n log n) 的时间复杂度和常数空间复杂度。

首先找到中间节点，然后使用归并排序就可以了。

### LeetCode 19. Remove Nth Node From End of List

### LeetCode 82. Remove Duplicates from Sorted List II

### LeetCode 142. Linked List Cycle II

### LeetCode 2. Add Two Numbers

### LeetCode 143. Reorder List

### LeetCode 61. Rotate List

### LeetCode 138. Copy List with Random Pointer

### LeetCode 25. Reverse Nodes in k-Group

### LeetCode 23. Merge k Sorted Lists





