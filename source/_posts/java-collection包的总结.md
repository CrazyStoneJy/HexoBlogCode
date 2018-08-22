---
title: java collection包的总结
date: 2018-01-26 22:21:42
tags: [Java]
---


HashMap 初始容量为16，容量必须为2的指数，loadfactor为0.75F.当单链表上的元素大于8并且所有元素大于64时，转为红黑树.

Hashtable 本人觉的现在这个类比较鸡肋，普通存储不考虑并发直接使用HashMap即可，如有并发使用ConcurrentHashMap.
Hashtable起始容量为11,每次扩容为:2*capacity(+1.由于Hashtable没有使用数组加链表或者红黑树的方式，并且是个同步方法,所以他的效率不高,比较浪费空间,好在get方法时间复杂度是O(1)