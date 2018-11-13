---
layout: post_layout
comments: true
title: JAVA 源代码阅读
sub_title: 
meta-keyword: java, java source code, jdk source code
meta-description: java源代码阅读摘录
categories: java
tags: java-code
description: 本文主要记录了，在阅读java源代码的过程得出的心得体会，与摘录的部分源代码。
date: 2018-11-12
---

## HashMap

* `putVal` hash下标的确定`(tab.length-1) & hash`


    Node<K,V>[] tab; Node<K,V> p; int n, i;
    if ((tab = table) == null || (n = tab.length) == 0)
        n = (tab = resize()).length;
    if ((p = tab[i = (n - 1) & hash]) == null)
        tab[i] = newNode(hash, key, value, null);

由于tab.length为2的倍数，假设为1000，减一后，变成了0111，与hash做与运算后，得到0111，即为hash%tab.length.

* `resize()`扩容，原hash链表按低位与高位拆分为两组


    if ((e.hash & oldCap) == 0) {
        if (loTail == null)
            loHead = e;
        else
            loTail.next = e;
        loTail = e;
    }

    if (loTail != null) {
        loTail.next = null;
        newTab[j] = loHead;
    }
    if (hiTail != null) {
        hiTail.next = null;
        newTab[j + oldCap] = hiHead;
    }

由于，新容量为旧容量的2倍，因此，原hash值按新容量取余后，余数可能为原余数，也可能为原余数+旧容量长度。
举个简单的例子来说明，原长度为4，新长度为8；按长度为4来计算，余数为1的数有：1、5、9、13、17;如果按长度为8来计算，则对应的余数为：1、5、1、5、1。可以看得出，同一组数分别对4和8取余，分成了两组。

`(e.hash & oldCap) == 0`的判断，由于容量oldCap都是2的倍数(假设第k位为1)，当`e.hash & oldCap=0`时，则e.hash为oldCap的偶数倍(则k位不为1)+余数，当`e.hash & oldCap=1`时，则e.hash为oldCap的奇数倍(则k位为1)+余数。

数学水平不够，只能按照这样子来理解了。

* `split`分拆
    

    if (lc <= UNTREEIFY_THRESHOLD)
        tab[index] = loHead.untreeify(map);
    else {
        tab[index] = loHead;
        if (hiHead != null) // (else is already treeified)
            loHead.treeify(tab);
    }


当hiHead为null时，实际上，整棵树的结构没有任何改变，所以低位的loHead依旧是原来的树的根节点。之前怎么想都想不通，后来，突然就豁然开朗了。



