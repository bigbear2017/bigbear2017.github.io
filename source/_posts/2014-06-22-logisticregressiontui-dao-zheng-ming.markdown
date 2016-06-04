---
layout: post
title: "LogisticRegression算法介绍"
date: 2014-06-22 21:43:48 +0800
comments: true
categories: 机器学习 基础算法
---
##### 版权声明：本文所有内容都是原创，如有转载请注明出处，谢谢。

这是我要复习和介绍的第一个机器学习算法，logistic regression，既可以用来作回归，
也可以用来作分类。训练比线性回归复杂一点，但比SVM容易很多，可以说是学习机器学习
入门的算法。以下是Logistic Regression模型的推导证明，主要参考的是
Elements of statistical learning, data mining, inference and prediction。
我会用几个系列讨论这个问题，以及相关的优化算法。

## 模型定义
暂且先讨论只有两种输出的情况, $ y = 0/1 $, 假设输入向量为$m$维, $ x \in R^{m} $。
模型的定义如下：

$$ \log \frac{ Pr(y = 1 \vert x ) } { Pr(y = 0 \vert x )} = \beta_0  + \beta $$

由上面的式子可以得到：

$$ \frac{Pr( y = 1 \vert x )} { Pr( y = 0 \vert x  ) } = \exp^{\beta_0 + \beta x } $$

并且知道:

$$ Pr( y = 1 \vert x ) + Pr( y = 0 \vert x ) = 1 $$

把式子代到上面得到：

$$ Pr(y = 1 \vert x ) = \frac{ \exp^{ \beta_0 + \beta x } } { 1+ \exp^{ \beta_0 + \beta x } } \
Pr( y = 0 \vert x ) = \frac{ 1 } { 1+ \exp^{ \beta_0 + \beta x  } } $$

## 最大化Log-Likelihood
假设 $ p = Pr(y = 1 \vert x ) $ 并且 $ \beta x = \beta_0 + \beta x $, Log-Likelihood如下：

$$ \ell( \beta ) = \Sigma_{ i = 1 }^{ N } \{  y_i \log(p) + (1-y_i) \log(1-p)  \} $$

$$ = \Sigma_{i = 1}^{N} \{ y_i \log(p) + log(1-p) - y_i \log(1-p) \} $$

$$ = \Sigma_{i = 1}^{N} \{ y_i \beta^{T} x_i - \log ( 1 + \exp^{ \beta x_i} ) \} $$
上面这个函数是什么意思呢？先从似然函数和最大似然估计说起。
似然函数的意思是，如果变量$X^{n}=(x_1, x_2, ... , x_n )$的联合分布，服从概率密度函数
$p(x_1, x_2, ... , x_n, \theta )$，那么

$$ L(\theta) = p( x_1, x_2, ... , x_n, \theta ) $$

因此我们可以把数据中的$x$和$y$，放在一起看成一个$m+1$维的变量，如果$y=1$，根据上面的公式
$L(\theta) = p( x, 1, \theta )$，也就是预测的值为1的概率。如果$y=0$，根据上面的公式，
$L(\theta) = p( x, 0, \theta )$，就是预测的值为0的概率。因此把所有样子的似然估计加起来，
就得到了上面的式子。这个最大似然估计法，是统计中最常用的方法之一。

为了最大化上面的函数，我们可以使用Newton Method来进行迭代，
初始化一个$\beta$，然后不断地更新$\beta$，最终得到最优解，
我会在下一篇里面介绍Newton Method，以及相关简单的优化算法。
根据Newton Method，我们需要计算方程的一阶导数$F^{1}(\beta)$和
二阶导数$F^{2}(\beta)$，一阶导数计算还是比较简单的，只要会一般
的求导公式，应该都可以得到下面的结果：

$$ F^{1}(\beta) = \Sigma_{i = 1}^{N} \{ y_i x_i - \frac{1}{ ( 1+\exp^{\beta x_i} ) } \exp^{ \beta x_i } x_i \}$$

$$ = \Sigma_{i = 1}^{N} x_i ( y_i - p_i ) $$

这个只要会求导数，应该这个公式都可以求出来。先来看看这个结果，
首先这个结果是一个向量，在迭代的过程中，当$\beta$为某一个值的时候，
$ y_i $ 和 $ p_i $都是一个实数。而$ x_i $是$m$维的，因此这个结果是一个向量。
这个也比较容易理解，对一个向量求导，结果也应该是一个向量。
但要求二阶导数就会麻烦一点，因为一阶导数已经是一个向量了，再对向量进行求一阶导数，
结果是一个$ N * N $的矩阵，这个矩阵叫Hessian Matrix。如果对这个方面有问题的话，
可以在google里面搜索: derevative of vectors。有一个pdf文档详细的介绍了，
如何去求向量的导数，可以看看理解一下。这里的推导还算简单，直接可以
把$x_i$看成一个一维的变量，推导如下：

$$ F^{2}(\beta) = F^{1} \{ \Sigma_{i=1}^{N} x_i ( y_i - \frac{\exp^{\beta x }}{(1 + \exp^{\beta x})} ) \} $$

$$ = F^{1} \{ \Sigma_{i=1}^{N} x_i ( y_i - \frac{1}{1 + \exp^{-\beta x } } ) \} $$

$$ = \Sigma_{i=1}^{N} - x_i x_i \frac{\exp^{-\beta x}}{(1 + \exp^{-\beta x })^{2} } $$

$$ = \Sigma_{i=1}^{N} - x_i x_i p_i ( 1 - p_i ) $$

再看看这个结果，$x_i$乘上$x_i$很显然是一个矩阵，而$p_i$和$1-p_i$都是常数。
这样我们就得到了一个函数的二阶导数。
然后就可以根据Newton Method来进行迭代了，迭代如下：

$$ \beta^{new} = \beta^{old} - \frac{F^{1}(\beta) }{F^{2}(\beta) } $$

因为$F^{2}(\beta)$是一个矩阵，除一个矩阵和除一个数是类似的，都是乘上它的$-1$次方，
就是要算这个矩阵的逆矩阵。

## 矩阵形式
这样的话，按照这个方程我们就可以训练LR的模型了。如果使用矩阵计算的话，可以写成下面的样子，
$ X $是输入矩阵，$Y$为标签向量，$P$为预测向量，$W$为对角线矩阵，值为 $ p_i ( 1-p_i ) $：

$$ F^{1}(\beta) = X^{T} ( Y - P ), F^{2}(\beta) = - X^{T} W X  $$

Newton Method的迭代为：

$$ \beta^{new} = \beta^{old} + ( X^{T} W X )^{-1} X^{T} (Y - P ) $$

##前进的方向
好，到此为止，logistic regression最简单的结果就有了。但是在实际的生活中，
却很少有人用Newton method去解logistic regression。因为要计算Hessian Matrix，
这个matrix太可怕了，如果是$m$维的输入，这个matrix就是$m*m$维的。
像现在训练的模型，动不动就上百万维的，再多内存也不够啊。
所以我们需要一些更好优化的方法，Conjugate gradient或者 LBFGS。
另外有的时候，我们的输入不一定就是0/1，如果有更多的类，怎么办呢？这个以后慢慢讨论。
