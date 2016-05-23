---
layout: post
title: "Newton Method, BFGS和LBFGS算法介绍"
date: 2014-09-12 21:49:55 +0800
comments: true
categories: 机器学习 基础算法
---

本文主要介绍Newton Method，BFGS, LBFGS 算法，最重要的是算法推导的过程，以及这三个算法的比较。

## Newton Method
假设$y = f(x)$，为了求方程$f(x) = 0$的解，我们可以用下面迭代的方式：

$$ x_{n+1} = x_n - \frac{ f(x_n) } { f^{\prime} (x_n) } $$

比如$ f(x) = x^{2} - 5 $，我们可以这么来求：

$$ x_1 = 5, f(5) = 20, f^{\prime}(5) = 10, x_2 = 5 - 20 / 10 = 3 $$
$$x_2 = 3, f(3) = 4, f^{\prime}(3) = 6, x_3 = 3 - 4/6 = 7/3 $$
$$x_3 = 7/3, f(7/3) = 4/9, f^{\prime}(7/3) = 14/3, x_4 = 7/3 - 2/21 = 47/21 $$

这样$ 47/21 $ 已经非常接近5的平方根了。为什么可以这么计算呢？证明如下：
根据泰勒展开式，$ y = f(x) $ 在 $x_n$处的展开为：

$$ y = f^{\prime}(x_n) ( x_{n+1} - x_n ) + f(x_n) $$

如果$ y = 0 $, 那么可得如下结果：

$$ x_{n+1} = x_n - f(x_n) / f^{\prime}(x_n) $$

在许多机器学习的算法中，我们经常可能碰到的是另外一种形式，比如我们想最大化log likelihood函数。
这个时候，我们会对函数先求一个导，导数$f^{\prime}(x) = 0$的地方一般就是极值点。因此这个时候，我们想求的是
$ f^{\prime}(x) = 0 $，而不是$ f(x) = 0 $。如果求$ f^{\prime}(x) = 0 $，则方程的解如下：

$$ x_{n+1} = x_n - \frac{f^{\prime}(x)}{f^{\prime \prime}(x)} $$

但是因为这个方法要求函数的二阶导数，就像在logistic regression中说的一样。求出来的Hessian Matrix实在太大，
求逆也非常的麻烦。因此就有人想到了更为简单的算法，LBFGS就是其中一个比较好的方法，我们先从BFGS算法说起。

## BFGS
BFGS算法是由Broyden, Fletcher, Goldfarb 和 Shanno四个人共同提出的。这里我们把他们的名字列出来，纪念一下。:)
由最简单的Line Search的方法，我们知道对于求方程$ f(x) = 0 $的解，我们可以进行如下迭代：

$$ x_{k+1} = x_k + \alpha_k \rho_k $$

其中 $\alpha_k$ 为步长，$\rho_k$为迭代方向。根据泰勒展开，我们可以把$f(x)$写成下面格式：

$$ f(x\_{k+1}) = f(x_k) + f^{\prime}(x_k)(x\_{k+1} - x_k) + \frac{1}{2} (x\_{k+1} - x_k)^{T} f^{\prime \prime}(x_k) ( x\_{k+1} - x_k ) $$

上面的函数，也可以写成$ \rho $ 的函数，如下：

$$ m\_k(\rho) = f_k + f^{\prime T}\_k \rho + \frac{1}{2} \rho^{T} B_k \rho $$

其中$f_k$为$f(x_k)$, $f^{\prime}\_k$为$f(x)$在$x_k$处的导数。根据上面的式子，当$\rho = 0$的时候，

$$ m\_k(0) =  f_k, m^{\prime}\_k (0) = f^{\prime T}\_k $$

因此在任何一点$x_k$，$f(x_k)$都可以写成$m_k$的形式，方程的曲线也应该是相同的。根据上面的定义，我们可以得到方程在
$x\_{k+1}$处，$m\_{k+1}$的表达式为:

$$ m\_{k+1}(\rho) = f\_{k+1} + f^{\prime T}\_{k+1} \rho + \frac{1}{2} \rho^{T} B_{k+1} \rho $$

根据这个表达式，$m_{k+1}$ 这个表达式在$x_k$处的导数为:

$$ f^{\prime}(x_k) = m^{\prime}\_{k+1}(-\alpha_k \rho_k) = f^{\prime T}\_{k+1} - \alpha_k B\_{k+1} \rho_k $$

所以得到：

$$ B\_{k+1} \alpha_k \rho_k = f^{\prime}\_{k+1} - f^{\prime}_k $$

为了简化，定义如下:

$$ s_k = x\_{k+1} - x_k = \alpha_k \rho_k, y\_{k} = f^{\prime}\_{k+1} - f^{\prime}\_k $$

所以上面的结果可以写成:

$$ B_{k+1} s_k = y_k $$

假设$ H_{k+1} $为$B_{k+1}$的逆矩阵，那么我们可以得到:

$$ H_{k+1} y_k = s_k $$

利用$H_k$，$H_{k+1}$可以通过下面的方式获得:

$$ \min_H || H - H_{k+1} || $$

subject to $ H = H^{T}, Hy_k = s_k $.这样我们可以得到$ H_{k+1} $的解为：

$$ H\_{k+1} = (I - p_k s_k y^{T}_k)H_k(I-p_k y_k s^{T}\_k) + p_k s_k s^{T}\_k$$

其中$p_k = \frac{1}{y^{T}_k s_k}$.到这里BFGS的解法就算明白了。相对于Newton Method，BFGS方法不用
计算Hessian Matrix的逆矩阵。但是它仍然要计算一个matrix，所以内存的占用仍然是一样的。具体算法如下：
```
while || f^{\prime}_k|| > \epsilon
          compute search direction
                  p_k = -H_k f^{\prime}_k
          x_{k+1} = x_k + \alpha_k p_k where \alpha_k is computed from a line search
              procedure to satisfy the Wolfe conditions.
          Define s_k = x_{k+1} - x_k and y_k = f^{\prime}_{k+1} - f^{\prime}_k
          Compute H_{k+1} with above formula.
          k = k + 1
end( while )
```

### LBFGS
根据上面的推导，BFGS算法可以写成下面的格式:

$$ x\_{k+1} = x\_{k} - \alpha\_k H\_k f^{\prime}\_k(x) $$

其中$\alpha_k$为步长，$H_k$可以写成下面的格式:

$$ H\_{k+1} = V^{T}\_k H\_{k} V_k + p_k s_k s^{T}\_k $$

其中$p\_k = \frac{1}{y^{T}\_k s_k}$, $V_k = I - p_k y_k s^{T}\_k $, and $s_k = x\_{k+1} - x_k$,$y_k = f^{\prime}\_{k+1} - f^{\prime}\_{k}$.

由于计算$H\_{k+1}$相当的麻烦，我们可以存储$m$个最近的$y_i, s_i$，然后来近似当前的$H\_{k+1}$. 根据$H\_{k+1}$的定义，我们不难发现这是一个递归的定义。在所有计算的过程中，其实就只有$H_0$这么一个矩阵，其他的全部是向量。如果我们使用一个对角矩阵来近似$H_0$，那么所有的计算都是可以用向量来表示。这个也是为什么它叫limited memory BFGS。
```
calculate the new direction \rho_k
q = f^{\prime}(x_k)
for i = k-1 ... k-m
  \alpha_i = p_i s^T_i q
  q = q - \alpha_i y_i
end for
r = H_0 q
for i = k-m ... k-1
  \beta = p_i y^T_i r
  r = r + s_i(\alpha_i - \beta)
end for
return r which is H_{k}*f^{\prime}_k
```

## L-BFGS algorithm
```
choose starting point x_0, integer m > 0;
k = 0;
repeat
      Choose H_0
      Compute \rho_k = - H_{k}*f^{\prime}_k
      Compute x_{k+1} = x+k + \alpha_k \rho_k, where \alpha_k is the step length
          which statisfy the Wolfe conditions.
      if k > m
              Discard the vector pair{ s_{k-m}, y_{k-m} } from storage.
      Compute and save s_k = x_{k+1} - x_{k}, y = f^{\prime}_{k+1} - f^{\prime}_k
      k = k+1
```
参考资料
http://en.wikipedia.org/wiki/Newton%27s_method
Numerical Optimization, Chapter 6, 7.
