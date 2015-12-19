---
layout: post
title: "如何使用MapReduce计算文本的tf-idf"
date: 2014-06-13 17:26:18 +0800
comments: true
categories: 算法
---

本文介绍如何使用MapReduce进行文本的tf-idf计算。
程序的输入是一个个的文本，类似下面的：

doc1: w1, w2, w3 

doc2: w4, w5, w6 

输出为一个个的Vector，可以是Sparse的Vector，或者是Dense Vector,
我们就假设使用Sparse的Vector，输出如下：

doc1_id, 1:0.1, 2:0.1, 3:0.03 

doc2_id, 4:0.1, 5:0.1, 6:0.01 

## 如何使用MapReduce进行计算呢？
对一个单词$w_i$, 在文档$D_j$中，tf-idf的计算公式如下：
$$ tf_idf= \frac{ N(w_i) } { N( D_j ) } * \log \frac{ \vert D \vert } { \vert j: w_i \in D_j \vert + 1 } $$
其中$N(w_i)$ 是$w_i$在$D_j$中的频率，$N(D_j)$是$D_j$中所有词的频率，也就是文本的长度，
$|D|$为所有文本的个数，$| j: w_j \in D_j |$ 为包含$w_i$的文档个数。

统计tf-idf需要两个MapReduce Job，一个job统计每个词的tf-idf，然后将每个文档中的词聚合到一起。
```
MapReduce1 Mapper Input:
doc1: w1, w2, w3 
doc2: w4, w5, w6
MapReduce1 Mapper Output:
w1_index: (doc1_id, tf_w1, w1_index)
```
其中 doc1_id根据对文档名字进行hash得到，tf_w1根据doc1中w1出现的次数来计算，
w1_index根据词典计算，context的counter统计文档数量|D|

MapReduce1 Reducer Output:

doc1_id: (w1_index, tf-idf )

tf-idf根据tf, |D|，所有包含w1的文档的个数

MapReduce2 Mapper: Identity Mapper

MapReduce2 Reducer: 根据doc_id，将所有相同文档归并到一起，得到:

doc1_id: w1_index:tf-idf, w2_index:tf-idf

doc2_id: w4_index:tf-idf, w5_index:tf-idf

还有另外一种方法是，先计算每个词的idf，然后再将idf做为distributed cache
分发给每一个mapper，然后得到每个文本的tf-idf.

