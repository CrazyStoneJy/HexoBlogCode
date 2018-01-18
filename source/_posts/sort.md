---
title: 常用排序算法Java语言实现
date: 2017-06-06 12:39:04
tags: [算法]
category: [Algorithm]
---

## **冒泡排序**
#### 算法原理:
>
1.比较相邻的元素。 如果第一个比第二个大，就交换他们两个。
2.对每一对相邻元素作同样的工作，从开始第一对到结尾的最后一对。 在这一点，最后的元素应该会是最大的数。
3.针对所有的元素重复以上的步骤，除了最后一个。
4.持续每次对越来越少的元素重复上面的步骤，直到没有任何一对数字需要比较。
<!-- more -->
#### 代码实现：

```java
    public int[] bubbleSort(int[] array){
        for(int i=0;i<array.length-1;i++){
            for(int j=0;j<array.length-1-i;j++){
                if(array[j]>array[j+1]){
                    int temp = array[j+1];
                    array[j+1]=array[j];
                    array[j]=temp;
                }
            }
        }
        return array;
    }
```
#### 上面代码是没有经过优化的代码，如果我们想进行优化，那该怎么优化呢？因为上面的代码每次都会去比较前一个值和后一个值的大小，当我们发现在比较的过程中有一组值没有交换，则说明该元素之后的元素都是有序的，所以该元素之后的比较次数其实是多余，因此，我们增加一个bool变量标识在比较的过程中，是否有元素进行了交换，代码如下。
#### 改良后的冒泡排序代码实现：

```java
 public int[] bubbleSortPro(int[] array){
        boolean isContinue=true;
        for(int i=0;i<array.length-1&&isContinue;i++){
            isContinue=false;
            for(int j=0;j<array.length-1-i;j++){
                if(array[j]>array[j+1]){
                    int temp = array[j];
                    array[j]=array[j+1];
                    array[j+1]=temp;
                    isContinue=true;
                }
            }
        }
        return array;
    }
```
#### 时间复杂度
O(n2)

## **选择排序**
#### 算法原理：
>
> 1.在长度为N的无序数组中，第一次遍历n-1个数，找到最小的数值与第一个元素交换；
 2.第二次遍历n-2个数，找到最小的数值与第二个元素交换；
 3.第n-1次遍历，找到最小的数值与第n-1个元素交换，排序完成。

与冒泡排序的区别：
> 与冒泡排序不同的是选择排序不是每次发现数据小的时候就会交换位置，而是将位置的下标记录下来，比较完一轮后，再去交换位置。

#### 代码实现：

```java
public static int[] selectionSort(int[] array) {
        for (int i = 0; i < array.length - 1; i++) {
            int min = i;
            for (int j = i + 1; j < array.length; j++) {
                if (array[j] < array[min]) {
                    min = j;
                }
            }
            if (min != i) {
                int temp = array[i];
                array[i] = array[min];
                array[min] = temp;
            }
        }
        return array;
    }
```
#### 时间复杂度
O(n2)

## **插入排序**
#### 算法原理：
> 1.从第一个元素开始，该元素可以认为已经被排序;
   2.取出下一个元素，在已经排序的元素序列中从后向前扫描;
   3.如果该元素（已排序）大于新元素，将该元素移到下一位置;
   4.重复步骤3，直到找到已排序的元素小于或者等于新元素的位置;
   5.将新元素插入到该位置后;
   6.重复步骤2~5。

#### 代码实现：

```java
public static int[] insertionSort(int[] array) {
        for (int i = 0; i < array.length - 1; i++) {
            for (int j = i + 1; j >= 0; i--) {
                if (array[j] < array[j - 1]) {
                    int temp = array[j];
                    array[j] = array[j - 1];
                    array[j - 1] = temp;
                } else {
                    break;
                }
            }
        }
        return array;
    }

```
