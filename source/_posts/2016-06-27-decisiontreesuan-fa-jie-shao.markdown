---
layout: post
title: "DecisionTree算法介绍--附代码实现"
date: 2016-06-27 23:16:03 +0800
comments: true
categories: 机器学习
---

## 算法介绍

决策树作为一种简单的算法，用途还是很广泛的。不仅仅这样，他还能够成为其他算法的基础。比如Random Forest，GBDT, AdaBoosting，等等。在实现上，也是非常的简单，训练的速度也是非常的快。先让我们直观上来感受一下，什么是决策树：

![decision tree](https://upload.wikimedia.org/wikipedia/commons/c/c6/Manual_decision_tree.jpg)

##模型训练

#### Regression Trees

从上面，我们知道一棵决策树，是由不同的节点构成的。从上到下，样本被不断地分到不同节点中去，一直到叶子节点。最终叶子上的节点，决定了这个叶子节点的预测值或者预测标签是什么。所以具体的过程如下：

1. 假设对于任意的分割变量 $j$ 和分割值$v$， 输入$X$可以被分成：

$$ R_1(j,v)  = \\{ X | X_j < v \\} \text{ and } R_2(j , v) = \\{ X | X_j > v \\} $$ 

2. j，v可以利用下面的方式确定：

$$\min\_{j,v} [ \sum\_{x_i \in R_1} ( y_i - c_1)\^2 + \sum_{x_i \in R_2} ( y_i - c_2)\^2 ] $$

其中: $ c_1 = avg( y_i | x_i \in R_1) $, $c_2 = avg(y_i | x_i \in R2) $



#### 伪代码

决策树的想法很简单，就是利用特征构成一颗树，树的左边是子树或者类别，每个节点就是决策点。那问题来了，如何找到每个决策点？稍微想一下的话，可能不难明白：

```
while height < MaxHeight and currentLevel has Nodes to split
  for each Node in current level : 
    for each feature and possible split in current node:
        find the best split s;
        split the node n according to s;
        if the left node or right node can not be split:
           continue
       split current Node into Left and Right
       put left and right into nextLevelList
  currentLevel = nextLevel
  height = height + 1
```

#### Classfication Trees

分类树主要是计算criterion的时候，有些差别。其他的总体上，也没什么变化。

Missclassification Error: $ \sum\_{k=1}\^K ( 1- p\_{mk} ) $ 

Gini Index: $\sum\_{k=1}^K p\_{mk}( 1- p\_{mk} )$

Cross-entropy: $\sum\_{k=1}\^K p\_{mk} \log p\_{mk}$

## 算法实现

这些天没有更新blog，就自己实现了一个简单的decision tree。它真的很简单，什么都没有。没有剪枝，没有优化地选feature。10w的samples，100维的数据20秒左右可以训练好。还有很多可以优化的地方。

code repository: [skywalker](https://github.com/bigbear2017/skywalker)
