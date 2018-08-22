---
title: kotlin语法总结
date: 2018-07-29 13:58:31
tags: [keolin]
categories: [kotlin]
---

#### 匿名内部类

```kotlin

interface IFoo{
    fun foo():Int;
    fun test();
} 

// 声明
fun setCallback(foo:IFoo){

}

// 调用
setCallback(object:IFoo{
    override fun foo():Int{

    }
    override fun test(){
        
    }
})

```