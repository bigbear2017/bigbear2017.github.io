---
layout: post
title: "Word2Vec算法介绍一: A Neural Probabilistic Language Model"
date: 2016-11-22 08:00:10 +0800
comments: true
categories: 机器学习 深度学习
---

##### 版权声明：本文所有内容都是原创，如有转载请注明出处，谢谢。

### 引言
这篇文章基本上是word2vec的开篇之作，后来的工作也都是在这篇文章的基础上，进行的优化。之所以要做word2vec，目的也是比较明显的。
在没有word2vec之前，如果我们想计算两个句子的相似性，就只能用ngram这种方式来计算相似性。但是这种方式的缺点是，由于词汇量都很大，导致生成的数据向量的维度都很大，并且很稀疏。另外，无法考虑两个词的语义相似性。比如，一只猫正在卧室里走。通过对语料的训练，我们应该可以得到，相似的语句像一只狗正在卧室里走。Word2Vec利用统计语言模型，对上面的问题有了一个初步的解决方法。

### 算法结构
word2vec基本上可以分为这样几步：

1. 对于词典中的每一个词，都关联一个特征向量
2. 使用特征向量来表示词组序列的概率函数
3. 利用词组数据来学习 特征向量和概率函数的参数

第一步很简单，对于每一个词，我们随机初始化一个特征向量。第二步，这个就是神经网络的设计了。具体的设计，会在下面说明。
第三步，就是通过数据训练神经网络，得到一个合理的特征向量和神经网络的参数。这个就比较简单了。先是用FeedForward计算输出值，再用BackForward 求导计算。

通过上面的方式，我们可以得到两个部分，一个是被压缩的特征向量。另外一个是训练好的神经网络。我们经常使用的是特征向量，因为通过它，我们会发现一些非常有趣的事情。比如，向量(猫) 和 向量(狗)会比较接近。甚至 向量(皇后) + 向量(男人) 和向量(皇帝)很接近。通过这种把词语转换成向量的方式，我们就可以做很多事情。比如聚类，分类，甚至是文档的匹配等等。

### 具体的过程
训练的过程是为了最大化log-likelihood，定义如下：

$$ L = \frac{1}{T} \sum_t \log f(w_t, w\_{t-1}, ..., w\_{t-n+1}; \theta ) + R(\theta) $$

论文中给出的具体计算过程如下图：

![word2vec](http://www.bigbear2017.com/images/word2vec-network.png =500x500)

从上面的图我们可以知道：

$$ f(x) = b + Wx + U tanh(d + Hx)$$

where $tanh(x) = \frac{e\^x - e\^{-x}}{ e\^x + e\^{-x} }$ and $\frac{d}{dx} tanh = 1 - tanh\^2 x$
神经网络的输入是$w\_t, w\_{t-1}, ..., w\_{t-n +1}$，通过mapping matrix C，我们得到每个词相应的vector $C(w\_t )$。每一次训练的时候，把输入$w\_{t-1}, ... , w\_{t-n+1}$，合并成一个vector，得到我们输入:

$$ x = ( C(w\_{t-1}), C(w\_{t-2}), ..., C(w\_{t-n+1})) $$

而我们输出层$y_i$是由$w_t$映射到 one-hot encoder得到。但是我们的神经网络的结果可能会大于1。所以作者就在最后一层增加了一个soft max.

$$ P( w\_t | w\_{t-1}, ..., w\_{t-n +1} )  = \frac{\exp\^{y\_{w\_t}}}{\sum\_{i} \exp\^{y_i}} $$

这样，我们就可以训练整个神经网络了。

#### 计算过程：
主要的forward过程比较简单，可以很容易完成。首先我们可以把$f(x)$ 分成两步：

$$a = d + Hx;$$

$$f(x) = b + Wx + U tanh(a)$$

back propagation需要计算偏导数会有点麻烦，具体如下：

$$ \frac{\partial f(x) }{\partial b} = 1$$

$$ \frac{\partial f(x)}{\partial W} = x$$

$$ \frac{\partial f(x)}{\partial U} = a$$

$$ \frac{\partial f(x)}{\partial d} =  \frac{\partial f(x)}{\partial a}  \frac{\partial a}{\partial d} = U ( 1 - a\^2) * 1$$

$$ \frac{\partial f(x)}{\partial H} =  \frac{\partial f(x)}{\partial a}  \frac{\partial a}{\partial H} = U ( 1 - a\^2) * x $$

#### 一个问题：如何训练C
 我们知道神经网络可以训练参数，这个很容易理解。但是C作为输入的一部分，如何进行训练呢？其实，应该说word2vec有两层的隐含层。C也是通过训练得到。每一次输入的时候，我们可以把C看成是$C = X$，这样，我们就可以对$C$进行求偏导，具体如下：

$$ \frac{\partial f(x)}{\partial C} =  \frac{\partial f(x)}{\partial a}  \frac{\partial a}{\partial X}  \frac{\partial X}{\partial C} = U ( 1 - a\^2) * H + W $$

### Word2Vec的意义
Word2Vec使我们很轻松的将一个词，转化成一个vector，类似于AutoEncoder，Word2Vec使我们可以很容易提取一些相对准确高维的特征。我个人觉得这也是它不同于其他语言模型的地方。像LDA，也可以提取一些高维的特征，但是效果明显不如Word2Vec。
