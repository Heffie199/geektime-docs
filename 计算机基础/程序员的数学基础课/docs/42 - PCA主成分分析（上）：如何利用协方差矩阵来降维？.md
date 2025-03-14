你好，我是黄申。

在概率统计模块，我详细讲解了如何使用各种统计指标来进行特征的选择，降低用于监督式学习的特征之维度。接下来的几节，我会阐述两种针对数值型特征，更为通用的降维方法，它们是**主成分分析PCA**（Principal Component Analysis）和**奇异值分解SVD**（Singular Value Decomposition）。这两种方法是从矩阵分析的角度出发，找出数据分布之间的关系，从而达到降低维度的目的，因此并不需要监督式学习中样本标签和特征之间的关系。

## PCA分析法的主要步骤

我们先从主成分分析PCA开始看。

在解释这个方法之前，我先带你快速回顾一下什么是特征的降维。在机器学习领域中，我们要进行大量的特征工程，把物品的特征转换成计算机所能处理的各种数据。通常，我们增加物品的特征，就有可能提升机器学习的效果。可是，随着特征数量不断的增加，特征向量的维度也会不断上升。这不仅会加大机器学习的难度，还会影响最终的准确度。针对这种情形，我们需要过滤掉一些不重要的特征，或者是把某些相关的特征合并起来，最终达到在减少特征维度的同时，尽量保留原始数据所包含的信息。

了解了这些，我们再来看今天要讲解的PCA方法。它的主要步骤其实并不复杂，我一说你就能明白，但是为什么要这么做，你可能并不理解。咱们学习一个概念或者方法，不仅要知道它是什么，还要明白是怎么来的，这样你就能知其然，知其所以然，明白背后的逻辑，达到灵活运用。因此，我先从它的运算步骤入手，给你讲清楚每一步，然后再解释方法背后的核心思想。

和线性回归的案例一样，我们使用一个矩阵来表示数据集。我们假设数据集中有m个样本、n维特征，而这些特征都是数值型的，那么这个集合可以按照如下的方式来展示。

![](https://static001.geekbang.org/resource/image/10/e2/10cf8973fd0f94e778c808bdda2881e2.png?wh=1296%2A344)

那么这个样本集的矩阵形式就是这样的：

![](https://static001.geekbang.org/resource/image/fb/20/fb10462182f9cc58b389771316f40720.png?wh=702%2A276)

这个矩阵是m×n维的，其中每一行表示一个样本，而每一列表示一维特征。让我们把这个矩阵称作样本矩阵，现在，我们的问题是，能不能通过某种方法，找到一种变换，可以降低这个矩阵的列数，也就是特征的维数，并且尽可能的保留原始数据中有用的信息？

针对这个问题，PCA分析法提出了一种可行的解决方案。它包括了下面这样几个主要的步骤：

1. 标准化样本矩阵中的原始数据；
2. 获取标准化数据的协方差矩阵；
3. 计算协方差矩阵的特征值和特征向量；
4. 依照特征值的大小，挑选主要的特征向量；
5. 生成新的特征。

下面，我们一步步来看。

### 1.标准化原始数据

之前我们已经介绍过特征标准化，这里我们需要进行同样的处理，才能让每维特征的重要性具有可比性。为了便于你回顾，我把标准化的公式列在了这里。

$x’=\\frac{x-μ}{σ}$

其中$x$为原始值，$u$为均值，$σ$为标准差，$x’$是变换后的值。需要注意的是，这里标准化的数据是针对同一种特征，也是在同一个特征维度之内。不同维度的特征不能放在一起进行标准化。

### 2.获取协方差矩阵

首先，我们来看一下什么是协方差（Covariance），以及协方差矩阵。协方差是用于衡量两个变量的总体误差。假设两个变量分别是$x$和$y$，而它们的采样数量都是$m$，那么协方差的计算公式就是如下这种形式：

![](https://static001.geekbang.org/resource/image/27/c2/2732d3255408c3bb4e01f6c2bd4499c2.png?wh=558%2A198)

其中$x\_k$表示变量$x$的第$k$个采样数据，$\\bar{x}$表示这$k$个采样的平均值。而当两个变量是相同时，协方差就变成了方差。

那么，这里的协方差矩阵又是什么呢？我们刚刚提到了样本矩阵，假设$X\_{,1}$表示样本矩阵$X$的第$1$列，$X\_{,2}$表示样本矩阵$X$的第$2$列，依次类推。而$cov(X\_{,1},X\_{,1})$表示第1列向量和自己的协方差，而$cov(X\_{,1},X\_{,2})$表示第1列向量和第2列向量之间的协方差。结合之前协方差的定义，我们可以得知：

![](https://static001.geekbang.org/resource/image/a3/a0/a3664cc303c7473df6dae33c6a0fbca0.png?wh=728%2A200)

其中，$x\_{k,i}$表示矩阵中第$k$行，第$i$列的元素。 $\\bar{X\_{,i}}$表示第$i$列的平均值。

有了这些符号表示，我们就可以生成下面这种协方差矩阵。

![](https://static001.geekbang.org/resource/image/ff/6c/ffe746718b2dde0a76051066326d226c.png?wh=1268%2A312)

从协方差的定义可以看出，$cov(X\_{,i},X\_{,j})=cov(X\_{,j},X\_{,i})$，所以$COV$是个对称矩阵。另外，我们刚刚提到，对于$cov(X\_{,i},X\_{,j})$，如果$i=j$，那么$cov(X\_{,i},X\_{,j})$也就是$X\_{,j}$这组数的方差。所以这个对称矩阵的主对角线上的值就是各维特征的方差。

### 3.计算协方差矩阵的特征值和特征向量

需要注意的是，这里所说的矩阵的特征向量，和机器学习中的特征向量（Feature Vector）完全是两回事。矩阵的特征值和特征向量是线性代数中两个非常重要的概念。对于一个矩阵$X$，如果能找到向量$v$和标量$λ$，使得下面这个式子成立。

$Xv=λv$

那么，我们就说$v$是矩阵$X$的特征向量，而$λ$是矩阵$X$的特征值。矩阵的特征向量和特征值可能不止一个。说到这里，你可能会好奇，特征向量和特征值表示什么意思呢？我们为什么要关心这两个概念呢？简单的来说，我们可以把向量$v$左乘一个矩阵$X$看做对$v$进行旋转或拉伸，而这种旋转和拉伸都是由于左乘矩阵$X$后，所产生的“运动”所导致的。特征向量$v$表示了矩阵$X$运动的方向，特征值$λ$表示了运动的幅度，这两者结合就能描述左乘矩阵$X$所带来的效果，因此被看作矩阵的“特征”。在PCA中的主成分，就是指特征向量，而对应的特征值的大小，就表示这个特征向量或者说主成分的重要程度。特征值越大，重要程度越高，我们要优先现在这个主成分，并利用这个主成分对原始数据进行变换。

如果你还是有些困惑，我会在下面一节，讲解更多的细节。现在，让我们先来看看给定一个矩阵，如何计算它的特征值和特征向量，并完成PCA分析的剩余步骤。我在下面列出了计算特征值的推导过程：

$Xv=λv$  
$Xv-λv=0$  
$Xv-λIv=0$  
$(X-λI)v=0$

其中I是单位矩阵。对于上面推导中的最后一步，我们需要计算矩阵的行列式。

![](https://static001.geekbang.org/resource/image/37/d0/373722f933c8a6afc97052c5f3686ed0.png?wh=1234%2A476)

$(x\_{1,1}-λ)(x\_{2,2}-λ)…(x\_{n,n}-λ)+x\_{1,2}x\_{2,3}…x\_{n-1,n}x\_{n,1}+…)-(x\_{n,1}x\_{n-1,2}…x\_{2,n-1}x\_{1,n})=0$

最后，通过解这个方程式，我们就能求得各种λ的解，而这些解就是特征值。计算完特征值，我们可以把不同的λ值代入$λE-A$，来获取特征向量。

![](https://static001.geekbang.org/resource/image/17/cb/1776be34cd7d73453a33e7259abbe0cb.png?wh=278%2A292)

### 4.挑选主要的特征向量，转换原始数据

假设我们获得了k个特征值和对应的特征向量，那么我们就有：

$Xv\_1=λ\_1v\_1$  
$Xv\_2=λ\_2v\_2$  
$…$  
$Xv\_k=λ\_kv\_k$

按照所对应的λ数值的大小，对这k组的v排序。排名靠前的v就是最重要的特征向量。

假设我们只取前k1个最重要的特征，那么我们使用这k1个特征向量，组成一个n×k1维的矩阵D。

把包含原始数据的m×n维矩阵X左乘矩阵D，就能重新获得一个m×k1维的矩阵，达到了降维的目的。

有的时候，我们无法确定k1取多少合适。一种常见的做法是，看前k1个特征值的和占所有特征值总和的百分比。假设一共有10个特征值，总和是100，最大的特征值是80，那么第一大特征值占整个特征值之和的80%，我们认为它能表示80%的信息量，还不够多。那我们就继续看第二大的特征值，它是15，前两个特征值之和有95，占比达到了95%，如果我们认为足够了，那么就可以只选前两大特征值，把原始数据的特征维度从10维降到2维。

## 小结

这一节，我首先简要地重温了为什么有时候需要进行特征的降维和基于分类标签的特征选择。随后，我引出了和特征选择不同的另一种方法，基于矩阵操作的PCA主成分分析。这种方法的几个主要步骤包括，标准化原始数据、获得不同特征的协方差矩阵、计算协方差矩阵的特征值和特征向量、选择最重要的主成分，以及通过所选择的主成分来转换原始的数据集。

要理解PCA分析法是有一定难度的，主要是因为两点原因：第一，计算的步骤有些复杂。第二，这个方法的核心思路有些抽象。这两点可能会让刚刚接触PCA的学习者，感到无从下手。

为了帮助你更好的理解，下一节，我会使用一个示例的矩阵进行详细的推算，并用两种Python代码进行结果的验证。除此之外，我还会分析几个要点，包括PCA为什么使用协方差矩阵？这个矩阵的特征值和特征向量又表示什么？为什么特征值最大的主成分涵盖最多的信息量？明白了这些，你就能深入理解为什么PCA分析法要有这些步骤，以及每一步都代表什么含义。

## 思考题

给定这样一个矩阵：

![](https://static001.geekbang.org/resource/image/96/9f/962b0abb078974d1d964627e43081f9f.png?wh=332%2A208)

假设这个矩阵的每一列表示一个特征的维度，每一行表示一个样本。请完成

1. 按照列（也就是同一个特征维度）进行标准化。
2. 生成这个矩阵的协方差矩阵。

欢迎留言和我分享，也欢迎你在留言区写下今天的学习笔记。你可以点击“请朋友读”，把今天的内容分享给你的好友，和他一起精进。
<div><strong>精选留言（12）</strong></div><ul>
<li><span>Joe</span> 👍（4） 💬（1）<p>一直有个问题为什么协方差是除以m-1，而不是m。方差，均方根等公式也是除m-1。好奇怪。</p>2019-03-22</li><br/><li><span>！null</span> 👍（2） 💬（1）<p>是不是只有方阵才能求特征值和特征向量？特征值和特征向量的个数与矩阵的维度有关系吗？特征向量的维度和矩阵的维度有关系吗？</p>2021-08-23</li><br/><li><span>yaya</span> 👍（2） 💬（1）<p>所以上只是讲解pca的步骤吗？非常赞同要明白他是为什么被提出的，怎么来的观点，但是pca如果只是记步骤很容易忘记，觉得还是从如何建模，然后推导而来更有印象。</p>2019-03-22</li><br/><li><span>牛奶</span> 👍（1） 💬（0）<p>请问，在解释协方差公式的时候，xbar是不是应该表示的是“m个样本x这个属性的平均值”?还有一个问题就是，关于这句“协方差是用于衡量两个变量的总体误差”，对于协方差的意义我只能理解到，协方差表示的是两个变量之间的相关性，为什么是两个变量的总体误差呢？</p>2021-03-12</li><br/><li><span>骑行的掌柜J</span> 👍（1） 💬（1）<p>这里Xn,1+...... 后面的括号是不是写错了？还是写掉了一部分？黄老师
(x1,1​−λ)(x2,2​−λ)…(xn,n​−λ)+x1,2​x2,3​…xn−1,n​xn,1​+…)−(xn,1​xn−1,2​…x2,n−1​x1,n​)=0
谢谢 因为有点没看懂😂</p>2020-07-06</li><br/><li><span>！null</span> 👍（0） 💬（1）<p>|(X-λI)|  那个竖线是什么意思？行列式是什么意思？怎么算的</p>2020-10-10</li><br/><li><span>qinggeouye</span> 👍（4） 💬（2）<p>markdown 语法支持不是很好

(1) 标准化原始数据
$$
x&#39; = \frac{x-μ}{σ}
$$
第一列 

均值 $μ_1 = 0$ , 方差 ${σ_1}^2 = [(1-0)^2 + (2-0)^2 + (-3-0)^2]&#47;3 = 14&#47;3$ 

第二列 

均值 $μ_2 =1&#47;3 $ , 方差 ${σ_2}^2 = [(3-1&#47;3)^2 + (5-1&#47;3)^2 + (-7-1&#47;3)^2]&#47;3 = 248&#47;9$ 

第三列 

均值 $μ_3 =-19&#47;3 $ , 方差 ${σ_3}^2 = [(-7+19&#47;3)^2 + (-14+19&#47;3)^2 + (2+19&#47;3)^2]&#47;3 = 386&#47;9$ 

则，
$$
\mathbf{X&#39;} = \begin{vmatrix} 0.46291005&amp;0.50800051&amp;-0.10179732\\0.9258201&amp;0.88900089&amp;-1.17066918\\-1.38873015&amp;-1.3970014&amp;1.2724665\\\end{vmatrix}
$$

(2)协方差矩阵
$$
\mathbf{cov(X_{,i}, X_{,j})} = \frac{\sum_{k=1}^m(x_{k,i} - \bar{X_{,i}})(x_{k,j} - \bar{X_{,j}})}{m-1}
$$

$$
\mathbf{X&#39;}.mean(asix=0) = [0,0, -7.401486830834377e-17]
$$

$$
\mathbf{cov(X_{,i}, X_{,j})} = \frac{(\mathbf{X&#39;[:,i-1]} - \mathbf{X&#39;[:,i-1]}.mean()).transpose().dot(\mathbf{X&#39;[:,j-1]} - \mathbf{X&#39;[:,j-1]}.mean())} {m-1}
$$

协方差矩阵(对角线上是各维特征的方差)：
$$
\mathbf{COV} = \begin{vmatrix} \mathbf{cov(X_{,1}, X_{,1})} &amp; \mathbf{cov(X_{,1}, X_{,2})} &amp; \mathbf{cov(X_{,1}, X_{,3})} \\ \mathbf{cov(X_{,2}, X_{,1})} &amp; \mathbf{cov(X_{,2}, X_{,2})} &amp; \mathbf{cov(X_{,2}, X_{,3})} \\ \mathbf{cov(X_{,3}, X_{,1})} &amp;\mathbf{cov(X_{,3}, X_{,2})} &amp;\mathbf{cov(X_{,3}, X_{,3})}\\\end{vmatrix} = \begin{vmatrix} 1.5 &amp; 1.4991357 &amp; -1.44903232 \\ 1.4991357 &amp; 1.5 &amp; -1.43503825 \\ -1.44903232 &amp; -1.43503825 &amp; 1.5 \\\end{vmatrix}
$$
</p>2019-03-31</li><br/><li><span>余泽锋</span> 👍（3） 💬（1）<p>import numpy as np
import pandas as pd
from sklearn.preprocessing import scale
array = np.array([[1, 3, -7], [2, 5, -14], [-3, -7, 2]])
array = scale(array)
df = pd.DataFrame(array)
df.corr()</p>2019-04-11</li><br/><li><span>建强</span> 👍（2） 💬（1）<p>1.标准化

第一步：计算每列平均值：
E(X1) = 0, E(X2) = 1&#47;3, E(X3) = -19&#47;3

第二步：计算每列方差：
D(X1) = 14&#47;3, D(X2) = 248&#47;9, D(X3) = 386&#47;9

第三步：根据标准化公式：[X - E(X)] &#47; SQRT[D(X)]做标准化处理，结果：

X11 = 0.4628,   X12 = 0.508,    X13 = -0.1018 
X21 = 0.9258,   X22 = 0.889,    X23 = -1.1707
X31 = -1.389,   X32 = -1,397,   X33 = 1.2725

2.协方差矩阵

[[  7.        ,  17.        , -20.5       ],
 [ 17.        ,  41.33333333, -49.33333333],
 [-20.5       , -49.33333333,  64.33333333]]</p>2020-11-01</li><br/><li><span>piboye</span> 👍（0） 💬（0）<p>最重要的数学推导没有啊</p>2023-12-17</li><br/><li><span>罗耀龙@坐忘</span> 👍（0） 💬（0）<p>茶艺师学编程

在隔壁《人人都能学会的编程入门课》找虐回来，继续找黄老师找虐。

而且今天作业又是和留言框过不去……

标准化：
[  0.378   0.577   0.083
   0.755   0.577  -0.956
  -1.133  -1.155  1.039  ]

协方差矩阵：
[      1      0.981     -0.934   
   0.981   0.9999   -0.804  
  -0.934  -0.804     1.014  ]</p>2020-05-09</li><br/><li><span>黄振宇</span> 👍（0） 💬（0）<p>有个问题，黄老师能否帮忙解答下。
降维后的特征集合是之前所有特征的子集合吗，是相当于是先对数据的特征向量做了筛选吗？只不过我们把筛选的工作交给了特征值？

还是说降维之后乘以新的特征向量之后，原始数据的意义是否变了呢？</p>2019-11-29</li><br/>
</ul>