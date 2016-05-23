---
layout: post
title: "SVM算法介绍"
date: 2015-05-22 19:35:43 +0800
comments: true
categories: 机器学习 基础算法
---

在没有深度学习之前，SVM非常火。最近似乎很少有人再用svm了，不过还是让我们来回顾一下这个经典的算法吧。

### 1. Perceptron 
说到svm，我们可以先从最简单的perceptron说起。Percetron是最简单的一种分类器。给n个点(X, Y), 假设
$X \in R\^p, Y \in (0, 1)$，如何得到一个分类器$f(x) = \beta * x + \beta_0$，使得所有被错误分类的点，到分类线的距离最小。我们希望说，如果被错误分类了，也不要错的太厉害。这样的话，我们就可以得到下面的式子:

$$
D( \beta, \beta_0 ) = - \sum_{i} y_i( \beta x_i + \beta_0 ) 
$$

根据gradient decent，随机初始化一个$\beta$，然后每次更新$\beta$如下：

$$ (\beta, \beta_0 )\_{n+1}= ( \beta, \beta_0 )\_n + \rho * (y_i x_i, y_i )$$

在扫描几遍数据之后，我们得到一个稳定的$\beta, \beta_0$就可以了。

### 2. Best Seperating Hyperplane
在上面求解的过程中，我们会发现，每次得到的解都是不一样的。随着初始化，数据的输入，都会发生变化。那怎么才是最好的seperating hyperplane呢？SVM提出了一个观点，任何一个类中的点到分类线的距离都是最大的，这个就是最好的。
所以从上面，我们可以得到:

$$ 
\begin{matrix}  
\max_{\beta  \ \ \beta_0 } M  \\\\
subject \ \ to \ \ \frac{y_i ( \beta x_i + \beta_0  )} {||\beta||} \ge M 
\end{matrix}  
$$

从上面的式子，我们知道，对于 $ || \beta || $，我们取任何值，它都不会影响点到线的距离，因为我们已经做了归一化了.那么我们就取$ || \beta || = \frac{1}{M} $ 好了, 这样我们可以得到:  

$$ 
\begin{matrix}  
\max_{\beta  \ \ \beta_0 }  \frac{1}{ ||\beta || } \\\\
subject \ \ to \ \ y_i ( \beta x_i + \beta_0  ) \ge 1
\end{matrix}  
$$

稍微转化一下，我们可以得到下面的式子:
$$ 
\begin{matrix}  
\min_{\beta  \ \ \beta_0 }  \frac{1}{2}{ ||\beta ||^2 } \\\\
subject \ \ to \ \ y_i ( \beta x_i + \beta_0  ) \ge 1
\end{matrix}  
$$

这样的话，我们就可以对上面的式子进行求解了，根据拉格朗日算法，我们可以得到: 

$$ 
L_P = \frac{1}{2}{ ||\beta ||^2 } - \sum_{ i = i}^{n} \alpha_i [ y_i ( \beta x_i + \beta_0  ) - 1 ]
$$ 

对$\beta, \beta_0$，分别求偏导，我们可以得到:

$$
\begin{matrix}
\frac{\partial L_P} {\partial \beta } = \beta - \sum\_{i = 1}^{n} \alpha_i y_i x_i = 0 \\\\
\frac{\partial L_P} {\partial \beta_0 } = \sum\_{i=1}^{n} \alpha_i y_i = 0
\end{matrix}  
$$

代入原来的式子，我们可以得到一个dual的形式:

$$
\begin{matrix}
L_D =  \sum_{i = 1}^{n} \alpha_i - \frac{1}{2} \sum\_{i=1}^{n} \sum\_{j=1}^{n} \alpha_i \alpha_j y_i y_j x\_i\^T x_j 
\\\\
subject \ \ to \ \ \alpha_i \ge 0 \ and \ \sum\_{i=1}^{n} \alpha_i y_i  = 0
\end{matrix}
$$

从上面的式子，我们可以看到：

1. 如果$\alpha_i >0 $，那么$y_i (\beta x_i + \beta_0) = 1$，那么点就在边界上面
2. 如果$\alpha_i = 0 $，那么点不在边界上面

### 3. 如何求解
经过上面的变换，我们可以得到一个dual，然后使用 SMO 算法，或者其他算法我们都可以得到解。但是都是比较麻烦的。我们可不可以像其他算法一样，使用最简单的Gradient Descent 来求解呢？如果可以的话，那样会简单很多。(待续)

