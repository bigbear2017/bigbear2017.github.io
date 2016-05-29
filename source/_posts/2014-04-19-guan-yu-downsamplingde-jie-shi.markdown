---
layout: post
title: "关于Downsampling的解释"
date: 2014-04-19 21:18:25 +0800
comments: true
categories: 机器学习 算法
---

### 为什么进行DownSampling?

在很多预测的场景，负例样本过多。比如有超过10亿以上。如果这些样本全部都拿来训练，训练的时间比较长，机器可能会撑不住。这个时候，我们不会使用全部的负例，而是使用1/100甚至1/1000的样本去训练。这样训练的结果就不是原本的概率，而是在down sampling之后的概率。这样就我们需要对两个概率进行矫正。



### 如何矫正?

既然使用现在的模型得到的概率, 不是在原来样本上得到的概率，那如何进行矫正呢？假设原本的概率为$P_1$, downsampling之后的概率为$P_2$，正例和负例分别为$P, N$，down sampling使用$1/t$的随机概率进行down sampling. 所以，我们可以得到：

$$ \begin{matrix} 
P\_1 = \frac{P}{P+N} \\\\
P\_2 = \frac{P}{P+ N/t}
\end{matrix}$$ 

由上面的式子，我们可以得到: 

$$ \begin{matrix} 
P = \frac{P_1} {1- P_1} N \\\\
P = \frac{P_2}{1- P_2} \frac{N}{t}
\end{matrix}$$

所以，我们可以得到: 

$$ P_2 = \frac{P_1 t }{P_1 t + 1 - P_1}$$

$$ P_1= \frac{P_2}{t - P_2 t + P_2}$$

假如我们使用的logistic regression，那么: $P_2 = \frac{1}{1 + \exp^{-wx}}$，代入得到:

$$ P_1 = \frac{1}{(1+\exp^{-wx}) t - t + 1}$$

$$ P_1 = \frac{1}{\exp^{-wx}t  + 1}$$

$$ P_1 = \frac{1}{\exp^{-wx+\ln{t}}  + 1}$$

这样，在我们训练的模型，之后再加一个bias，我们就得到了，一个样本在未downsampling之前的同等概率。



#### 其他应用

OverSampling方面其实也是类似。这方面还要再思考一下。
