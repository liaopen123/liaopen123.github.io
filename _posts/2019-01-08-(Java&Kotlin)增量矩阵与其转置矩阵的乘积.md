---
layout:     post
title:  (Java&Kotlin)增量矩阵与其转置矩阵的乘积
subtitle: 百度16年校招的面试题&AMCAT面试题
date:     2019-01-08
author:     Pony
header-img: img/bg-life-tour.jpg
catalog: true
tags:
    - 面试题
---
题不难，都是大学基础问题，但是昨天晚上做题的时候，发现网上只有C++的代码实现，并没有Java的，自己实现了，思路很清楚，测试结果也正确。
### 题目

>增量矩阵是一个元素为初始值initialValue的递增值的矩阵
  例如，如果初始值initialValue=1，且维度为rows=3 和 columns =3,
  则增量矩阵为：

 
 1  2  3
 
 4  5  6

 7  8  9

  写一个算法，将原始增量矩阵与其转置阵相乘。



>输入
  函数/方法的输入包括三个参数：一个表示初始值的正整数 initialValue，表示增量矩阵中行数的正整数，和表示增量矩阵中列树的正整数。
  输出
  返回由增量矩阵和其转置矩阵相乘得到的一个二维矩阵

示例
>输入：
  initialValue =1 
  rows = 3 
  columns = 3
   
    
  
  

输出：
 
  14  32  50
  
  32  77  122
  
  20  122 194
  
  解释  对于
   initialValue =1   rows = 3   columns = 3的情况，增量矩阵为
  

  
  1  2  3
  
  4  5  6
  
  7  8  9
  
  其转置矩阵为
  
  1  4  7
  
  2  5  8
  
  3  6  9
  
  因此，将由此产生的乘积矩阵为
  
  14  32  50
  
  32  77  122
  
  20  122 194
  


## 代码实现
### Java
```java
public int[][] transposeMultMatrix(int initialValue, int rows, int columns){
    int[][]  array =  getOrignalArray(initialValue, rows, cloumus);
        int[][] reveseArray = getReveseArray(array);
        int[][] plus = getPlus(array, reveseArray);
         return  plus;
}

  //获取增量矩阵
    private static int[][] getOrignalArray(
          int   initialValue,
            int rows,
             int cloumus
    ) {
        int firstValue = initialValue;
        int[][] array = new int[rows][cloumus];
        for (int i=0;i< rows;i++) {
            for (int j = 0; j< cloumus;j++) {
                array[i][j] = firstValue;
                firstValue++;
            }
        }
        return array;
    }


    /**
     * 得到转置矩阵
     * 转置矩阵的概念：行列互换即可
     */
    private static int[][] getReveseArray(int[][] array) {
        int rows =  array.length;
        int cols = array[0].length;
        int[][] reveseArray  = new int[cols][rows];
        for (int i=0;i< rows;i++) {
            for (int j = 0; j< cols;j++) {
                reveseArray[i][j] = array[j][i];
            }
        }
        return reveseArray;
    }


    /**
     * 增量矩阵和转置矩阵的乘积
     * 当矩阵A的列数等于矩阵B的行数时，A与B可以相乘。
     */

    private static int[][] getPlus(int[][] originalArray, int[][] reverseArray) {
        int oRows = originalArray.length;
        int oCols = originalArray[0].length;

        int rRows = reverseArray.length;
        int rCols = reverseArray[0].length;

        int[][] array = new int[oRows][rCols];
        if(oCols!=rRows){
            throw new RuntimeException("A矩阵的列与B矩阵的列不相等");
        }

        for (int i=0;i< oRows;i++) {
            for (int j = 0; j< rCols;j++) {
                int value =0;
                for (int m = 0; m< oRows;m++) {
                    value+=originalArray[i][m]*reverseArray[m][j];
                }
                array[i][j] = value;
            }
        }
        return array;
    }

}

```

### Kotlin

```kotlin
  fun getPlus(initialValue :Int,rows:Int,cloumus :Int): Array<IntArray> {
        val  array =  getOrignalArray(initialValue, rows, cloumus)
        val reveseArray = getReveseArray(array)
        var finalArray = transposeMultMatrix(array,reveseArray)

      return  finalArray
    }
    /**
     * 增量矩阵和转置矩阵的乘积
     * 当矩阵A的列数等于矩阵B的行数时，A与B可以相乘。
     */

    private fun transposeMultMatrix(originalArray: Array<IntArray>, reverseArray: Array<IntArray>): Array<IntArray> {
           val oRows = originalArray.size
            val oCols = originalArray[0].size

            val rRows = reverseArray.size
            val rCols = reverseArray[0].size

            var array = Array(oRows){IntArray(rCols)}
        if(oCols!=rRows){
            throw RuntimeException("A矩阵的列与B矩阵的列不相等")
        }

        for (i in 0 until oRows) {
            for (j in 0 until rCols) {
                var value =0
               for(m in 0 until oRows){
                       value+=originalArray[i][m]*reverseArray[m][j]
                }
                array[i][j] = value
            }
        }
        return array
    }

    /**
     * 得到转置矩阵
     * 转置矩阵的概念：行列互换即可
     */
    private fun getReveseArray(array: Array<IntArray>): Array<IntArray> {
       var rows =  array.size
       var cols = array[0].size
        var reveseArray  = Array(cols){IntArray(rows)}
        for(i in 0 until rows){
            for(j in 0 until  cols) {
                reveseArray[i][j] = array[j][i]
            }
        }
        return reveseArray



    }

    //获取增量矩阵
    private fun getOrignalArray(
        initialValue: Int,
        rows: Int,
        cloumus: Int
    ): Array<IntArray> {
        var firstValue = initialValue
        var array = Array(rows) { IntArray(3) }//二维数组
        for (i in 0 until rows) {
            for (j in 0 until cloumus) {
                array[i][j] = firstValue
                firstValue++
            }
        }
        return array
    }
```