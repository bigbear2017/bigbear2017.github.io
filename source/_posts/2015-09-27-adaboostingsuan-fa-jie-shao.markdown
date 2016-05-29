---
layout: post
title: "AdaBoosting算法介绍"
date: 2015-09-27 21:41:04 +0800
comments: true
categories: 基础算法 机器学习
---

Boosting 是一种很有意思的算法。在没有SVM之前，今天就让我们来一同回顾一下这个经典的算法吧。

## 1. 模型定义
假设模型的输入为$X \in R\^p$，输出为$ Y \in (-1, 1)$，输入和输出都有N个点。我们手上呢，有一个很简单的classfier，$G(x)$，他实在不怎么样，可能就比随机猜要好一点点。我们如何利用这个简单的分类器，对数据进行很好的分类呢？AdaBoosting就提出一种很好的办法: 将弱的分类器组合起来，然后我们形成一个强的分类器。这个想法很好，但是我们真的可以把分类器组合起来吗？首先怎么把分类器组合起来呢？直接用余数来训练吗？这样组合起来，分类器变的更糟糕，怎么办？AdaBoosting巧妙地利用了每一个分类器的优点，并且组合了起来。结果也是相当地惊人，在它刚刚被提出来的时候，几乎是当时最好的算法。首先，组合分类器的定义如下:

$$ G(x) = sign ( \sum_{i =1 }^{M} \alpha_m G_m(x)) $$

那如何得到每一个弱分类器的权重呢？AdaBoosting 是利用了每个分类器的错误率来进行组合的，每一次用$G(x)$来分类的错误率就定义如下:

$$ E = \frac{1}{N} \sum_{i=1}^{N} I(y_i \ne G(x_i)$$

## 2.模型训练
Algorithm AdaBoost 

>1.  Initialize  $ \omega_i = \frac{1}{N} $,  i = 1, 2, ... N
>2. For m = 1 to M:
> - (a) Fit a classifier G(x) to training data using weights $\omega_i$
> - (b)Compute 
> - $$ err_m = \frac{\sum\_{i=1}^{N} \omega_m I( y_i \ne G_m(x))} {\sum\_{i = 1}^{N} \omega_i}  $$
> - (c) Compute $ \alpha_m = \log(\frac{( 1 - err_m)} {err_m} ) $
> - (d) Update $\omega_i  = \omega_i \exp(\alpha_m I( y_i \ne G_m(x_i) )$
>3. Output  $ G(x) = sign[ \sum_{m=1}^{M} \alpha_m G_m(x) ] $

## 3. 算法解读

1. 为什么 $\alpha_m$定义是 $\log(\frac{( 1 - err_m)} {err_m} )$ ?

首先$err_m$的取值范围是$(0, 1)$。如果分类器$G_m(x)$的错误率越高，那么$err_m$就越接近于1, 否则就越接近于$0$。而根据常识来说，如果一个分类器很差劲，错误率越高呢，我们就希望他的权重就越低，越接近于0。而这样的话，就让我们想到了一个很常见的函数sigmod fuction。就让我们来证明这个来历吧。

$$ \begin{matrix} 
err_m =  \frac{1}{1 + \exp^{\alpha}} \\\\
=> 1 + \exp^{\alpha} = \frac{1}{err_m} \\\\ 
=> \exp^{\alpha} = \frac{1}{err_m} - 1 \\\\
=> \alpha = \log(\frac{( 1 - err_m)} {err_m} )
\end{matrix} $$

上面并不是logistic function(lf)，而是一个按照y轴对称的lf，因为如果直接用lf的话，err越大，权重也会越大了。

2. 为什么每次要更新$\omega_i$ ?
对于 $\omega_i$ 的更新，我们可以进行这样的解读:
>if $y_i == G(x_i)$ $\omega_i == \omega_i$
>else $\omega_i = \omega_i * \frac{1- err_m}{err_m}$ 

对于 $\frac{1- err_m}{err_m}$，我们知道$err_m$越大，值越小。也就是说，错误率越高，我们的就权重越低。这是可以理解的，如果$err_m$很大，说明分类器不好。被这样一个分类器给分错了，那我们可能觉得没关系。但是如果$err_m$很小，说明分类器很好，被一个很好的分类器分错了，说明这个点，我们希望在下次的时候，可以被其他的分类器补充。
