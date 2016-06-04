---
layout: post
title: "GradientBoosting算法介绍"
date: 2015-11-24 23:45:10 +0800
comments: true
categories: 基础算法 机器学习
---
##### 版权声明：本文所有内容都是原创，如有转载请注明出处，谢谢。 


### 算法介绍
由AdaBoosting算法，我们知道了什么是Boosting算法，算法大概的结构如下:

$$ G(x) = \sum_{i = 1}^{M} \alpha_i g_i( x ) $$

其中$g_i(x)$是弱分类器，$\alpha_i$是每个分类器的权重。在每一步中，AdaBoosting利用那些被分错类的点，然后增加权重，重新训练一个新的分类器，并且计算分类器的权重。
那什么是Gradient Boosting呢？类似于AdaBoosting, Gradient Boosting也是一种Boosting的算法，结构和上面一样。只是Gradient Boosting是利用Gradient来重新训练一个新的分类器，然后计算权重。

### 算法理解

##### 1. 简单的Boosting 
如果我们只有一个简单的分类器$g(x)$，不用AdaBoosting，如何进行Boosting呢？
可能最简单的是用Residual来进行训练，过程如下:

>for j = 1 to M

>- $ r\_{ji} = ( y_i - G\_{j-1}(x_i) ) $ //r_j should be a vector, size of N

>- fit $g\_j(x)$ to $r_j$ 

>- $\alpha\_j = \arg \min\_{\alpha} \sum\_{i = 1}^n L( y_i, G\_{j-1} ( x_i ) + \alpha * g_j(x_i) )  $

>- $G_j(x) = G_{j-1}(x) + \alpha_j g_j(x) $


##### 2. 使用Gradient 
上面的每一步，我们使用了一个Residual $(y_i - G\_{j-1}(x_i) )$ 来重新训练模型。这个表面上看是挺好的，每一次我们都在逼近$y$。但实际上，我们真的在减少Loss吗？假设我们使用Squared Error作为Loss. 那么: $L=  \sum\_{i=1}\^N ( y_i - \sum\_{j=1}^M g_j(x_i) )\^2$ 。这个是不是让我们想到Least Squared Regression呢？里面的$\beta$是如何更新的呢？因为是minimize loss，所以，我们是每次加上负梯度，更新如下:
$\beta\_{j+1} = \beta\_{j} - \frac{\partial L}{\partial \beta }$

看到这个，是不是我们好像似乎明白了什么。假如，我们把每一个$g_i(x)$都看成变量，如果我们想$G(x)$更接近$y$，那我们应该如何更新呢？
$G(x) = G (x) - \frac{\partial L}{\partial G(x)}$
从这个角度，我们再来理解$y_i - G_j(x_i)$的话，就可以知道它其实就是$L = (y - G(x))\^2$的负梯度($-\frac{\partial L} {\partial G(x)} = - ( y - G(x) ) * -1 = y - G(x) $).

#### 其他梯度
##### 1. Absolute Loss

Absolute loss的定义如下: 

$$ L(y, G) = |y - G|$$

梯度如下:
$$ \frac{\partial L}{\partial G} = 
\begin{cases} 
y - G(x) & if y \ge G(x) \\\\\\
G(x) - y & otherwise 
\end{cases} $$

##### 2. Huber Loss 

$$ L_\delta(y, f(x)) = 
\begin{cases}
 \frac{1}{2}(y - f(x))\^2                   & \textrm{for} |y - f(x)| \le \delta, \\\\\\
 \delta\, |y - f(x)| - \frac{1}{2}\delta\^2 & \textrm{otherwise.}
\end{cases} $$

所以
$$
\frac{\partial L_\delta(y, G(x))} {\partial G} = \begin{cases}
y - G(x)                 & \textrm{for} |y - G(x)| \le \delta, \\\\\\
 \delta sign( y - f(x) )  & \textrm{otherwise.}
\end{cases} $$

#### 算法过程
> Input: training set $\{(x_i, y_i)\_{i=1}\^n\}$

> 初始化learner function $G_0$为常数

> for j = 1 to M

>  '  '  $ r\_{ji} = -\frac{\partial L} {\partial G\_{j-1}(x_i)} $ //$r\_j$ should be a vector, size of N

>  '  '  fit $g_j(x)$ to $r_j$

>  '  '  $\alpha_j = \arg \min\_{\alpha} \sum\_{i = 1}\^n L( y_i, G_{j-1} ( x_i ) + \alpha * g_j(x_i) )  $

>  '  '  $G_j(x) = G_{j-1}(x) + \alpha_j g_j(x) $

初始化learner function $F_0$为常数
计算negative gradient
以negative gradient为目标进行训练得到一个简单的learner 
找到最优的常数得到新的更强的learner
