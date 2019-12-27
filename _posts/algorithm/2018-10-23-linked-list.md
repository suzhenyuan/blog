---

comments: true
title: 链表相关算法总结
sub_title: 
meta-keyword: 算法, 链表, algorithm, linked list, acm, leetcode
meta-description: 链表相关算法总结
categories: algorithm
tags: algorithm, linked list
description:  链表查找、插入、反转、删除、快慢指针、有环、相交等相关算法总结
date: 2018-10-23
---

## 快慢指针

   假设慢指针(走一步)用`()`表示，快指针(走两步)用`[]`表示。在起始位置与数量不同的情况下，快慢指针所处的位置如下：

* 第一种情况:起点都是在链表头

示例1：
[(1)] 2 3
1 (2) [3]

示例2：

[(1)] 2 3 4
1 (2) [3] 4
1 2 (3) 4 []

示例3：

[(1)] 2 3 4 5
1 (2) [3] 4 5
1 2 (3) 4 [5]

* 第二种情况:起点都是链表头的前一个位置

示例1：
[()] 1 2 3
    (1) [2] 3
    1 (2) 3 []

示例2：

[()] 1 2 3 4
    (1) [2] 3 4
    1 (2) 3 [4]

示例3：
[()] 1 2 3 4 5
    (1) [2] 3 4 5
    1 (2) 3 [4] 5
    1  2 (3) 4 5 []
 
* 第三种情况:慢指针的起点在链表头的前一个节点，快指针的起点在链表头

示例1：
1: () [1] 2 3
2:    (1) 2 [3]


示例1：
() [1] 2 3 4
    (1) 2 [3] 4
    1 (2) 3 4 []

示例3：
() [1] 2 3 4 5
    (1) 2 [3] 4 5
    1 (2) 3 4 [5]


从上面的例子可以看到，如果是把链表对半分的话，则可以采用第三种情况，慢指针的起点在链表头的前一个节点，快指针的起点在链表头。因为`slow.next`可以作为后半段的起点，设置一个新指针指向`slow.next`后，`slow.next` 设置为`null`即可。
如果是查找相交节点的话，那用第一种情况，快慢指针都是从链表头开始移动。





## 相交的判断：

1. 设置两个指针`Pa`和`Pb`分别指向给定的两个链表`headA`和`headB`
2. `Pa`与`Pb`同时向后移动, 到达列尾后，则指向另一个链表的头部，即`Pa`指向headB，`Pb`指向`headA`。
3. `Pa`与`Pb`到达链尾再回到另一个链表的表头后，此时两个指针相差的节点长度，刚好是两个链表相交节点前差异的部分，此时`Pa`与`Pb`同时向后移动，如果相同则存在相交，否则不存在相交的节点。


    public ListNode getIntersectionNode(ListNode headA, ListNode headB) {
        int times = 0;
        ListNode la = headA, lb = headB;
        while(null != la && null != lb){
            if(la == lb){
                return la;
            }
            la = la.next;
            lb = lb.next;
        }
        
        ListNode lab = null == la?lb:la;
        ListNode lt = null == la?headB:headA;
        ListNode ll = null == la?headA:headB;
        while(null != lab){
            lab = lab.next;
            lt  = lt.next;
        }
        
        while(null != ll && null != lt && ll != lt){
            ll = ll.next;
            lt = lt.next;
        }
        return ll ==lt?ll:null;
        
    }

