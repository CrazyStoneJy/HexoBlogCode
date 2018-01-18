---
title: java知识点总结
date: 2018-01-18 16:21:39
tags: [Java]
categories: [Java]
---

- 在实现一个对象的equals方法时，需要实现hashCode方法．Java中认为如果两个对象值相等,那么他们的hashCode必须一致，equals也必须相等．为什么要这麽做呢？因为:在Java中collection中HashMap,HashTable等，有Hash表的数据结构中，他们的key值的索引要使用对象的hashCode方法去进行Hash,如果他们的hashCode不一样，会被hash在不同的索引处，会被认为是不同的对象．当hashCode一致时，才比较他们的equals方法．所以，我们得出了结论，hashCode方法主要是为了存储在Hash表中定位索引值，因为在Java中Ｍap是一个非常常用的数据结构，使用他的子类一般会使用HashMap，因此，在Java中实现equals方法时，也实现hashCode的方法就成了一种规矩．