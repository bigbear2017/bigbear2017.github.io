---
layout: post
title: "Calibretion算法介绍"
date: 2016-06-01 23:59:03 +0800
comments: true
categories: 算法应用
---
##### 版权声明：本文所有内容都是原创，如有转载请注明出处，谢谢。

### 什么是Calibretion?
在进行点击率预估的过程中，我们发现这样一个现象。把所有点击率按照从小到大排序，然后每n个点击，计算copc（clicks over predicted clicks ），得到分段copc。如下图：
![copc1](http://7xv0xu.com1.z0.glb.clouddn.com/copc-1.png)
从上面的图，我们可以看到这样一个情况，在整条的曲线上，我们都会看到有时高估，有时低估。尤其在曲线的尾部，高估的严重，我们预估有10个点击，估计也就有3个点击左右。所以，我们需要进行矫正，让整条曲线都在1附近。

### 如何矫正
在进行矫正的时候，我们需要满足这样的条件:

1. 不可以改变目前预估的排序，因为如果改变了预估的排序，那么整个曲线的auc就会发生变化。
2. 保证矫正以后的copc在1附近。这样我们矫正才有意义。

解决的方式是我们使用Isotonic Regression 进行矫正。保序回归，刚刚好满足上面的要求。

### 什么是保序回归
简单地来说，保序回归的意思是这样，对于任意$n$个点$(a_1, a_2, ..., a_n)$, 找到n个点$(x_1, x_2, ... , x_n)$，使得$\sum\_{i=1}\^n (x_i - a_i)\^2$最小，并且对于任意的$i > j$，我们有$x_i \ge x_j$。比如下图中有100个点，从1到100个红色的点，蓝色是用linear regression解出来的线，绿色是保序回归解出来的线。
![isotonic-regression](http://7xv0xu.com1.z0.glb.clouddn.com/plot_isotonic_regression_1.png)

### PAVA (Pool Adjacent Violation Algorithm)算法
为了解决上面的问题，有人提出了PAVA算法。简单地来说，检查相邻两个block，还不能达到单调递增的状态，那么就将这两个block放在一起，合并成一个大的block。再对这个block，进行求解，之后递归上面的操作。
```
merge all values into blocks
solve each block, 
if the condition is not satisfied, repeat step 1 
```
用伪代码翻译下，可以得到下面这样的结果

```
输入: $ {(x_i, y_i)} $， 其中$x_i$为index，$y_i$为value
sort( {x_i, y_i} ) according to $x_i$
for each record (x_i, x_i, y_i) R
  merge Record R into RecordList L

def  mergeList( record R, recordList L ) :
    if L is empty:
      insert R to L and return
    get last record LR
    if LR.value < R.value:
      return 
    else:
       remove LR from L
       merge LR and  R to Rnew
       mergeList( Rnew, L )
```

PAVA 算法非常快，只需要O(n)的时间就好了，也非常有效。

#### 算法演示:
![pava-gif](http://7xv0xu.com1.z0.glb.clouddn.com/isotonic.gif)

### 纠正后的效果
如下图:
![copc2](http://7xv0xu.com1.z0.glb.clouddn.com/copc-2.png)
从上图，我们可以看到，整个曲线都在1附近。

### 代码实现
```
package com.skywalker.linear;

import com.google.common.collect.Lists;
import com.skywalker.utils.Sorter;
import com.skywalker.utils.Tuple;

import java.util.List;
import java.util.Stack;

/**
 * This class implements the PAVA algorithm.
 * @author caonn@mediav.com
 * @version 16/6/12.
 */
public class IsotonicRegression {
  private Stack<Record> records = new Stack<Record>();

  /**
   * This function will fit the data points according to pava algorithm.
   * @param tuples the input points, tuple.first is index, tuple.second is value
   */
  public  void fit (List<Tuple<Double, Double>> tuples) {
    Sorter.sortTupleList(tuples);
    for( Tuple<Double, Double> t : tuples) {
      Record record = new Record( t.first(), t.first(), t.second(), 1);
      merge(record);
    }
  }

  /**
   * Merge record to record list.
   * @param record record to be merged into records.
   */
  private void merge(Record record) {
    if( records.size() == 0 ) {
      records.add(record);
      return;
    }
    Record lastRecord = records.peek();
    if( lastRecord.value < record.value ) {
      records.add(record);
    } else {
      records.pop();
      int newNumber = record.number + lastRecord.number;
      Double newValue = ( record.number * record.value + lastRecord.number * lastRecord.value ) / newNumber;
      Record newRecord = new Record(lastRecord.lowIndex, record.highIndex, newValue, newNumber);
      merge(newRecord);
    }
  }

  public void printRecords() {
    for (Record record : records) {
      System.out.println(record.toString());
    }
  }

  public List<Double> predict(List<Double> indices) {
    List<Double> result = Lists.newArrayList();
    for( Double index : indices) {
      result.add(predict(index));
    }
    return result;
  }

  public double predict( double index ) {
    Record first = records.firstElement();
    if( index <= first.lowIndex ) return first.value;

    for(Record r : records) {
      if( index > r.lowIndex && index <= r.highIndex ) {
        return r.value;
      }
    }

    Record last = records.lastElement();
    return last.value;
  }
}

class Record {
  public final double lowIndex;
  public final double highIndex;
  public final double value;
  public final int number;
  public Record(double lowIndex, double highIndex, double value, int number) {
    this.lowIndex = lowIndex;
    this.highIndex = highIndex;
    this.value = value;
    this.number = number;
  }
  public String toString() {
    return String.valueOf(lowIndex) + " " + String.valueOf(highIndex) + " " + String.valueOf(value) + " " + String.valueOf(number);
  }
}
```
