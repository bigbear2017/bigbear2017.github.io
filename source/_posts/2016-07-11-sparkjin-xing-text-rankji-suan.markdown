---
layout: post
title: "Spark进行Text Rank计算"
date: 2016-07-11 21:47:08 +0800
comments: true
categories: spark
---

关于Text Rank具体的算法介绍，可以参考[Text Rank生成关键字和摘要](http://my.oschina.net/letiantian/blog/351154)。虽然有很多的python的实现，但是目前还没有spark的实现版本。Python的实现我试着使用了一下，如果在数据量很大的情况下，跑起来会很慢很慢。基本上是很难用的，所以就自己写了一个spark的版本。

我做这个的目的是，为了提取一段文字中的关键词组，而不是提取一段文字中重要的句子。不过我相信这两个方法是非常相似的：1. 提取关键词组，我们就需要计算词之间的关系； 2. 生成摘要，我们就需要句子之间的关系。具体的步骤如下:

#### 1. 读取数据
```
//load input file, It should have the following format:
//word1 word2 word3 word4 ...
//word2 word1 word4 word3 ...
JavaRDD<String> lines = sc.textFile(input);
```

#### 2. 生成所有相关联的pair
```
JavaPairRDD<String, Iterable<String>> links = lines.flatMapToPair(new PairFlatMapFunction<String, String, String>() {
 @Override
 public Iterable<Tuple2<String, String>> call(String line) {
   String[] parts = line.split("\t");
   List<Tuple2<String, String>> results = Lists.newArrayList();
   for (int i = 0; i < parts.length - 1; i++) {
     for (int j = i + 1; j < parts.length; j++) {
       results.add(new Tuple2<>(parts[i], parts[j]));
       results.add(new Tuple2<>(parts[j], parts[i]));
     }
   }
   return results;
 }
}).distinct().groupByKey().cache();
```

#### 3. 初始化rank
```
JavaPairRDD<String, Double> ranks = links.mapValues((Iterable<String> s1) -> 1.0);
```

#### 4. 不断迭代
```
for (int iter = 0; iter < maxIter; iter++) {
 JavaPairRDD<String, Double> contribs = links.join(ranks).values()
   .flatMapToPair(new PairFlatMapFunction<Tuple2<Iterable<String>, Double>, String, Double>() {
     @Override
     public Iterable<Tuple2<String, Double>> call(Tuple2<Iterable<String>, Double> s) throws Exception {
       int wordCount = Iterables.size(s._1());
       List<Tuple2<String, Double>> results = new ArrayList<>();
       for (String n : s._1()) {
         results.add(new Tuple2<>(n, s._2() / wordCount));
       }
       return results;
     }
   });
 ranks = contribs.reduceByKey((Double d1, Double d2) -> d1 + d2).
   mapValues((Double sum) -> dampingFactor + (1 - dampingFactor) * sum);
}
```
这样我们就可以得到每个词的rank。然后根据词的rank，我们找到一组最大rank的连续词组。这样我们就可以提取出句子中的关键词组了，效果看起来还可以。
