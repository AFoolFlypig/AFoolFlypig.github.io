---
title: SVM
date: 2020-08-09 21:27:15
categories: 机器学习
mathjax: true
tags: 
- SVM
- 机器学习
---
## 1. 模型定义
<!--more-->
支持向量机（support vector machines, SVM）是一种二分类模型，它的基本模型是定义在特征空间上的**间隔最大的线性分类器**，间隔最大使它有别于感知机。

给定训练数据集$D = \{(x_i,y_i)\}_{i = 1}^{N}$，其中$x_i \in \mathbb{R}^{p}$，$y_i \in \{-1, 1\}$，并且假设该训练数据集是线性可分的。SVM学习的基本想法是求解能够正确划分训练数据集并且几何间隔最大的分离超平面。
在样本空间中，划分超平面可通过如下线性方程来描述：
$$
w^{T}x + b = 0 \tag{2.1}
$$
其对应的分类决策函数为：
$$
f(x) = sign(w^{T}x + b) \tag{2.2}
$$
其中的$sign$为符号函数。

样本空间中任意点$x$到超平面$(w,b)$的距离可写为:
$$
r = \frac{\vert w^{T}x + b \vert}{\Vert w \Vert} \tag{2.3}
$$
假设超平面$(w,b)$能够将训练样本正确分类，即对于$(x_{i}, y_{i}) \in D$，若$y_{i} = +1$，则有$w^{T}x_{i} + b > 0$；若$y_{i} = -1$，则有$w^{T}x_{i} + b < 0$。令
$$
\begin{cases}
w^{T}x_{i} + b \geq +1, \quad y_{i} = +1 \\
w^{T}x_{i} + b \leq -1, \quad y_{i} = -1 \tag{2.4}
\end{cases}
$$
如下图所示，距离超平面最近的这几个训练样本点使式(2.4)的等号成立，他们被称为“支持向量”（support vector），两个异类支持向量到超平面的距离之和为
$$
\gamma = \frac{2}{\Vert w \Vert} \tag{2.5}
$$
它被称为“间隔”（margin）。

![](SVM\svm.png)

欲找到具有“最大间隔”（maximum margin）的划分超平面，也就是要找到能满足式(2.4)中约束的参数$w$和$b$，使得$\gamma$最大。对于式(2.4)，可以将其改写为如下的形式：
$$
y_{i}(w^{T}x_{i} + b) \geq 1 \tag{2.6}
$$
那么我们的优化目标就可以写作：
$$
\begin{align}
& \mathop{max}\limits_{w,b} \ \frac{2}{\Vert w \Vert} \tag{2.7} \\
& s.t. \ y_{i}(w^{T}x_{i} + b) \geq 1, \quad i = 1,2,\dots,N.
\end{align}
$$
显然，为了最大化间隔，仅需最大化$\Vert w \Vert^{-1}$，这等价于最小化$\Vert w \Vert^{2}$，即最小化$w^{T}w$，因此式(2.7)可以改写为：
$$
\begin{align}
& \mathop{min}\limits_{w,b} \ \frac{1}{2} w^{T}w \tag{2.8} \\
& s.t. \ y_{i}(w^{T}x_{i} + b) \geq 1, \quad i = 1,2,\dots,N.
\end{align}
$$
这就是支持向量机的基本型。

## 2. 模型求解

对式(2.7)使用拉格朗日乘数法构造拉格朗日函数得：
$$
L(w,b,\lambda) = \frac{1}{2} w^{T}w + \sum_{i = 1}^{N} \lambda_{i} [1 - y_{i}(w^{T}x_{i} + b)] \tag{2.9}
$$
其中$\lambda_{i} \geq 0$。

可以得到式(2.8)与下式等价：
$$
\begin{align}
& \mathop{min}\limits_{w,b} \, \mathop{max}_{\lambda} \ L(w,b,\lambda) \tag{2.10} \\
& s.t. \ \lambda_{i}\ge 0
\end{align}
$$
至于为什么式(2.8)与式(2.10)等价，下面进行一些解释。
当样本点满足式(2.8)中的约束条件，即在可行解区域内时，有：
$$
\mathop{max}_{\lambda} \ L(w,b,\lambda) = \frac{1}{2} w^{T}w
$$
当样本点不满足式(2.8)中的约束条件，即在可行解区域外时，有：
$$
\mathop{max}_{\lambda} \ L(w,b,\lambda) = +\infty
$$
综上，可以看出式(2.8)与式(2.10)是等价的。

利用强对偶性可将式(2.10)转化为如下的形式：
$$
\begin{align}
&\mathop{max}_{\lambda}  \, \mathop{min}\limits_{w,b} \ L(w,b,\lambda) \tag{2.11} \\
& s.t. \ \lambda_{i}\ge 0
\end{align}
$$
为了求解式(2.11)，需要先求得$L(w,b,\lambda)$对$w$和$b$的极小，再求对$\lambda$的极大。

令$L(w,b,\lambda)$对$w$和$b$求偏导，并使其为0可得：
$$
\begin{align}
w &= \sum_{i = 1}^{N} \lambda_{i} y_{i}x_{i} \tag{2.12} \\
0 &= \sum_{i = 1}^{N} \lambda_{i}y_{i} \tag{2.13}
\end{align}
$$
将以上两个式子代入(2.9)中得：
$$
\begin{align}
L(w,b,\lambda) &= \frac{1}{2} w^{T}w + \sum_{i = 1}^{N} \lambda_{i} [1 - y_{i}(w^{T}x_{i} + b)] \\
&= \frac{1}{2} w^{T}w + \sum_{i = 1}^{N} \lambda_{i} - \sum_{i = 1}^{N} \lambda_{i} y_{i}w^{T}x_{i} - \sum_{i = 1}^{N} \lambda_{i} y_{i}b \\
&= \frac{1}{2}(\sum_{i = 1}^{N} \lambda_{i} y_{i}x_{i})^{T} (\sum_{j = 1}^{N} \lambda_{j} y_{j}x_{j}) - \sum_{i = 1}^{N} \lambda_{i} y_{i} (\sum_{j = 1}^{N} \lambda_{j} y_{j}x_{j})^{T}x_{i} + \sum_{i = 1}^{N} \lambda_{i} \\
&= \frac{1}{2}\sum_{i = 1}^{N} \sum_{j = 1}^{N} \lambda_{i}\lambda_{j} y_{i}y_{j} x_{i}^{T}x_{j} - \sum_{i = 1}^{N} \sum_{j = 1}^{N} \lambda_{i}\lambda_{j} y_{i}y_{j} x_{i}^{T}x_{j} + \sum_{i = 1}^{N} \lambda_{i} \\
&= -\frac{1}{2}\sum_{i = 1}^{N} \sum_{j = 1}^{N} \lambda_{i}\lambda_{j} y_{i}y_{j} x_{i}^{T}x_{j} + \sum_{i = 1}^{N} \lambda_{i}
\end{align}
$$
则式(2.11)可以改写为：
$$
\begin{align}
&\mathop{min}\limits_{\lambda} \ \frac{1}{2}\sum_{i = 1}^{N} \sum_{j = 1}^{N} \lambda_{i}\lambda_{j} y_{i}y_{j} x_{i}^{T}x_{j} - \sum_{i = 1}^{N} \lambda_{i} \tag{2.14} \\
&s.t. \ \lambda_{i} \ge 0, \quad \sum_{i = 1}^{N} \lambda_{i}y_{i} = 0
\end{align}
$$
对于式(2.4)，我们可以使用序列最小优化算法（sequential minimal optimization，SMO）求解$\lambda$，然后根据$\lambda$求解出$w$和$b$。

式(2.12)已经给出了$w$的求解公式，那么$b$该怎么求呢？我们可以选取任意一个支持向量$(x_k,y_k)$，该支持向量满足$y_{k}(w^{T}x_{k} + b) = 1$，那么我们可以根据该式得到b的表达式：
$$
b = y_{k} - \sum_{i = 1}^{N} \lambda_{i} y_{i}x_{i}x_{k} \tag{2.15}
$$
在实际应用中，我们可以利用全体支持向量求解的平均值来表示$b$。

## 3. 算法实现

由于我对SVM中理解的不是太深入，对其中的一些数学细节一知半解，因此我暂时写不出SVM的实现代码。SVM算法的推导过程中涉及到许多最优化方面的知识，比如说对偶问题、KKT条件以及凸二次规划问题的求解，其他的机器学习算法中也或多或少的涉及到一些最优化方面的知识。因此我打算先系统的学习一下最优化基础理论，掌握一些数学工具之后，再去学习机器学习算法。

**参考资料**：
[拉格朗日函数为什么要先最大化？](https://zhuanlan.zhihu.com/p/55279698)
[白板推导：支持向量机](https://www.bilibili.com/video/BV1aE411o7qd?p=28)
[支持向量机（SVM）——原理篇](https://zhuanlan.zhihu.com/p/31886934)
[【机器学习】支持向量机 SVM（非常详细）](https://zhuanlan.zhihu.com/p/77750026)
[SVM原理介绍](https://blog.csdn.net/baidu_36557924/article/details/79517365)
[SVM（支持向量机)长文逐步理解](https://zhuanlan.zhihu.com/p/57648645)
周志华 《机器学习》