---
layout: post
title: "NeronNetwork算法介绍"
date: 2015-06-22 20:05:00 +0800
comments: true
categories: 机器学习 基础算法
---
##### 版权声明：本文所有内容都是原创，如有转载请注明出处，谢谢。 


最近深度学习实在太火了，火到各个公司都在搞实验室。但是深度学习的基础，还是从neron network，BP，这样的算法开始。
就简单回顾一下最简单的NN吧。

### 1. 算法介绍
我们假设只有一层的隐藏层，并且大小为M，输入$X \in R\^p$，输出$Y \in R\^K$，输出$Y$属于$K$ classes。所以，我们可以得到下面的定义:

$$ \begin{matrix} 
D_m = \rho ( \alpha_m0 + \alpha_m\^T X ) , m = 1, 2, ... M \\\\
T_k = \beta_k0 + \beta\_k\^T D, k = 1, 2, ... K \\\\
f_k(X) = g_k( T ) , k = 1,2, ... K 
\end{matrix}$$

其中$D = (D_1, D_2, ... , D_M)$, $\rho = \frac{1}{ 1 + \exp^{(-v)} }$是sigmod function, $g_k$可以是identity function，或者是softmax function $\frac{exp^{T_k}}{\sum_{l = 1}^{K} exp^{T_l}}$.

### 2. 求解
训练采用的back propagation的方式，就是先利用参数计算结果，然后根据结果计算梯度，根据梯度再优化参数。
根据上面的定义，我们可以得到loss function $R(\alpha, \beta)$如下:

$$
R(\alpha, \beta) = \sum\_{k = 1}^K \sum\_{n = 1}^N ( y_{ik} - f_k( x_i ) ) ^2 
$$

其中$N$是样本数, $K$是样本的class的数目. 所以对$\alpha, \beta$分别求偏导，我们可以得到:

$$\begin{matrix} 
\frac{\partial R_i}{\partial \beta\_{km} } = -2(y\_{ik} - f_k( x_i ) ) g^{\prime}(T_k) D\_{mi}   \\\\
\frac{\partial R_i}{\partial \alpha\_{mp}} =  -\sum\_{k=1}\^{K}2(y\_{ik} - f_k( x_i ) ) g^{\prime}(T_k) \beta\_{k}\rho\^{\prime}(\alpha\_m\^T x_i) x\_{ip}
\end{matrix}
$$

根据上面的式子，我们就可以利用gradient descent的方式，来训练模型了.

$$ \begin{matrix}
\beta\^{r+1}\_{km} = \beta\^{r}\_{km}  - \gamma_r \frac{\partial R_i}{\partial \beta\_{km} } \\\\
\alpha^{r+1}\_{mp} = \alpha^{r}\_{mp}  - \gamma_r \frac{\partial R_i}{\partial \beta\_{mp} } 
\end{matrix}$$

### 3. 向量化
上面的式子，实在看着太繁琐了，我们还是对其进行向量化，可能看着更简单，实现也更方便一些.
待续

### 4. 注意点
#### 1. 输入的起始点，不可以全部是零。因为全部是零的话，就永远全部是零，无法训练。也不可以过大，结果会不太好。最好的是随机靠近0的数。
#### 2. 隐藏层多一点比较好，可以表达的非线性的能力就越强一点
#### 3. 加正则防止过拟合。
