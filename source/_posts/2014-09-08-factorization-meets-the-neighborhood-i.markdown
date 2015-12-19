---
layout: post
title: "Factorization Meets the Neighborhood I"
date: 2014-09-08 21:54:12 +0800
comments: true
categories: 机器学习 算法
---
<!--more本文主要介绍Factorization Meets the Neighborhood中的算法一:baseline method，并简单地进行实现。我会尝试把几个算法都实现一下，然后分几个文章进行介绍。 Baseline estimates 假设一个user对一个item的评价为-->
本文主要介绍Factorization Meets the Neighborhood中的算法一:
baseline method，并简单地进行实现。我会尝试把几个算法都实现
一下，然后分几个文章进行介绍。

## Baseline estimates
假设一个user对一个item的评价为$r\_{ui}$, baseline estimate用$b\_{ui}$,
表示，那么baseline estimate可以理解如下:

$$ b_{ui} = \mu + b_u + b_i $$

其中$b_u$表示user评分的偏差，$b_i$表示item被评分的偏差。

比如我们想预测用户joe对Titanic的评分。假设总体上的平均分为3.7分，而
Titanic比其他电影要好，所以它总体上比平均分高0.5分。另外用户joe是一个
相当挑剔的用户，喜欢给低分，总体上比平均分低0.3分。因此，预测的值就是
3.7 - 0.3 + 0.5 = 3.9分。
因此我们就可以解下面的最小平方和问题：

$$ \displaystyle \min_b \displaystyle \sum\_{(u,i) \in K} ( r\_{ui} - \mu - b_u - b_i )^{2} + \lambda_1 ( \displaystyle \sum_u b^{2}\_u  + \displaystyle \sum\_{i} b^{2}_i ) $$

Linear Regression
如果看到上面的公式，不得不让我们想到线性回归中的Ridge。
Ridge的定义和上面很相似，定义如下:

$$ \displaystyle \min\_{\beta} \displaystyle \sum\_{i=1}^{N} ( y_i - \beta x_i )^{2} + \lambda_2 \sum \beta^{2}_j$$

和上面的方程比较，不难看出这两个方程非常的相似。假设所有用户的 $b_u$放在一起是一个向量$B_u$，另外所有item的$b_i$放在一起是一个向量$B_i$。我们只要把$\beta$看成$B_u$和$B_i$两个向量的连接。$y_i$看成是$r_{ui} - \mu$。而每个$x_i$则有两个1，下表为$b_u$和$b_i$在向量中的下标。
Ridge 的解如下：

$$ \beta = (X^{T}X + \lambda I )^{-1} X^{T} y $$

这样所有的$b_u$和$b_i$就解出来了。

## Statistical Method
有人提出说可以根据当前值，直接统计出$b_u$和$b_i$:

$$ b_i = \frac{ \sum\_{u \in R(i)} ( r_{ui} - \mu ) } {\lambda_2 + |R(i)|} $$

$$ b_u = \frac{ \sum\_{i \in R(u)} ( r_{ui} - \mu - b_i )   } {\lambda_3 + |R(u)|} $$

R(u)为用户u所有给出的评分，R(i)为项目i所有得到的评分。证明如下：
假设 $ y\_{ui} = r\_{ui} - \mu $，所以上面那个最小平方和的方程对$b_u$求偏导，可以得到：

$$ F^{1} (b_u) =  \sum\_{(u,i) \in K} 2( y_{ui} - b_u - b_i ) * (-1) + 2 \lambda_1 * b_u $$

如果想方程值最小，上面的偏导应该等于0。所以可以得到：

$$ \sum\_{i \in R(u)} y_{ui}  - |R(u)| * b_u - \sum\_{i \in R(u)} b_i =  \lambda_1 * b_u $$

$$ b_u = \frac{ \sum\_{i \in R(u)} ( y_{ui} - b_i ) } {\lambda_1 + |R(u)|} $$

同理，$b_i$也可以得到上面的式子。刚刚开始的时候，可以假设所有的$b_u$都为0, 然后求出$b_i$, 然后循环迭代。
基本上，迭代几轮就能达到上面最优的效果了。所以我们现在知道，别人提出的那个式子，只是一个开始步骤。如果
想得到理想的效果，可以稍微增加一些迭代次数。
具体实现
```
import numpy as np
'''
In this script, we will test the baseline method in [1].
We assume there are 100 users and 100 movies. Each user
will rate 30 movies. We will get a model to estimate how
much ratings a user will give for a movie
'''
seeds = np.random.normal( 3.7, 3.0, 100 )
ratings = []
for seed in seeds:
  ratings.append( np.random.normal( seed, 1.0, 100 ) )
ratings = np.vstack( ratings )
ratings[ ratings &lt; 0.01 ] = 0.01
unknown_index = np.random.random_integers( 0, 100, (100,100) )
non_zeros = unknown_index &lt;= 30
rate_num = np.sum( non_zeros )
x = np.zeros( ( rate_num, 200 ) )
y = np.zeros( rate_num )
lenx, leny = non_zeros.shape
count = 0
for i in range(lenx):
  for j in range( leny ):
    if non_zeros[i,j] == True:
      x[ count, i ] = 1
      x[ count, i+j+1 ] = 1
      y[ count ] = ratings[i, j]
      count += 1
mu = np.sum( y ) / rate_num
y = y - mu
xt = x.T
xtx = xt.dot( x ) + np.identity(200) * 5
xtx = np.linalg.inv( xtx )
beta = xtx.dot( xt ).dot( y )
'''
calculate beta using statistical method
according to this link:
http://semocean.com/因式分解实现协同过滤-及源码实现/
'''
beta2 = np.zeros( 200 )
for i in range( 100 ):
  count = 0
  for j in range( 100 ):
    if non_zeros[i, j] == True:
      beta2[i] += ratings[i,j] - mu
      count += 1
  beta2[ i ] /= ( count + 20 )
for i in range( 100 ):
  count = 0
  for j in range( 100 ):
    if non_zeros[ j, i ] == True:
      beta2[ i+100 ] += ratings[ j, i ] - mu - beta2[j]
      count += 1
  beta2[ i+100 ] /= ( count + 30 )
'''
calculate the mse
'''
mse1, ase1 = 0, 0
mse2, ase2 = 0, 0
mse3, ase3 = 0, 0
for i in range(lenx):
  for j in range(leny):
    if non_zeros[i,j] == False:
      error = ratings[i, j] - mu - beta[i] - beta[i+j+1]
      ase1 += np.abs( error )
      mse1 += error * error
      error2 = ratings[i, j] - mu
      ase2 += np.abs( error2 )
      mse2 += error2 * error2
      error = ratings[i, j] - mu - beta2[i] - beta2[i+j+1]
      ase3 += np.abs( error )
      mse3 += error * error
pre_num = 100 * 100 - rate_num
mse1 = np.sqrt(mse1 / pre_num )
mse2 = np.sqrt(mse2 / pre_num )
mse3 = np.sqrt(mse3 / pre_num )
ase1 = ase1 / pre_num
ase2 = ase2 / pre_num
ase3 = ase3 / pre_num
print 'prediction number: ', pre_num, 'mse1: ', mse1, 'ase1: ', ase1
print 'prediction number: ', pre_num, 'mse2: ', mse2, 'ase2: ', ase2
print 'prediction number: ', pre_num, 'mse3: ', mse3, 'ase3: ', ase3
```
测试了一下，效果比平均数要好上不少。刚刚开始的时候，我直接用一个正态分布来生成数据，
发现预测的竟然还不如平均数好。后来改变了数据生成的方法，baseline预测的看起来就正确了。

Baseline

 - prediction number:  6852
 - mse1:  1.98610991138
 - ase1:  1.60351615809

平均数

 - prediction number:  6852
 - mse2:  3.00171330689
 - ase2:  2.50388966193

Statistical Baseline

 - prediction number:  6852
 - mse3:  1.99252697533
 - ase3:  1.58618085221
