# 线性回归

线性回归模型非常简单，但是有句话说的好，简单即有效。

线性回归本来是统计学里的概念，但是经常在机器学习中应用非常广泛，当年读书时教授告诉我，线性回归也被称为"模型之母"，是许多强大的模型的基础。

线性回归 建模速度快，计算速度快，并且有很强的可解释性。但是不能很好的拟合非线性数据，所以要预先判断变量之间是否是线性关系。

现行回归所能模拟的关系远不止线性关系，"线性"指的是系数的线性，通过对特征值的非线性变换，已经线性模型的推广，输出和特征之间的函数关系也可以是非线性的。另一方面，线性模型的易解释性，使得它在物理，经济，商学等领域占据了重要的地位。

## 概述

在统计学中，线性回归（英语：linear regression）是利用称为线性回归方程的最小二乘函数对一个或多个自变量和因变量之间关系进行建模的一种回归分析。这种函数是一个或多个称为回归系数的模型参数的线性组合。只有一个自变量的情况称为简单回归，大于一个自变量情况的叫做多元回归（multivariable linear regression）。

在线性回归中，数据使用线性预测函数来建模，并且未知的模型参数也是通过数据来估计。这些模型被叫做线性模型。[2]最常用的线性回归建模是给定X值的y的条件均值是X的仿射函数。不太一般的情况，线性回归模型可以是一个中位数或一些其他的给定X的条件下y的条件分布的分位数作为X的线性函数表示。像所有形式的回归分析一样，线性回归也把焦点放在给定X值的y的条件概率分布，而不是X和y的联合概率分布（多元分析领域）。

### 用途

1. 如果目标是预测或者映射，线性回归可以用来对观测数据集的和X的值拟合出一个预测模型。当完成这样一个模型以后，对于一个新增的X值，在没有给定与它相配对的y的情况下，可以用这个拟合过的模型预测出一个y值。
2. 给定一个变量y和一些变量X1,...Xp，这些变量有可能与y相关，线性回归分析可以用来量化y与Xj之间相关性的强度，评估出与y不相关的Xj，并识别出哪些Xj的子集包含了关于y的冗余信息。

线性回归模型经常用最小二乘逼近来拟合，但他们也可能用别的方法来拟合，比如用最小化“拟合缺陷”在一些其他规范里（比如最小绝对误差回归），或者在桥回归中最小化最小二乘损失函数的惩罚。相反，最小二乘逼近可以用来拟合那些非线性的模型。因此，尽管“最小二乘法”和“线性模型”是紧密相连的，但他们是不能划等号的。

## 理论

给定数据集D={(x1, y1), (x2, y2), ... }，我们试图从此数据集中学习得到一个线性模型，这个模型尽可能准确地反应x(i)和y(i)的对应关系。这里的线性模型，就是属性(x)的线性组合的函数，可表示为：

$$
y = w_1x_1 + w_2x_2 + ... +w_dx_d + b
$$

使用向量可以表示为：
$$
y_i = w_i^Tx + b
$$

其中的 w=(w1,w2,w3,...wd) 表示列向量，这里的 w(weight) 是权重，表示对应的属性在预测结果的权重，权重越大，对于结果的影响越大，是线性模型的参数，用于计算结果。

通常线性回归模型的建立，就是如何求得变量参数的过程。根据求得的参数，我们可以对新的输入来计算预测的值。（也可以用于对训练数据计算模型的准确度）

我们根据已有数据集来求得属性的参数（相当于求得函数的参数），然后根据模型来对于新的输入或者旧的输入来进行预测（或者评估）。

## 一元线性回归

一元线性回归就是只拟合一个维度的参数与结果的关系。

对于数据集D={(x1, y1), (x2, y2), ... }，我们需要根据每组输入(x, y)来计算出线性模型的参数值，那么如何计算？
$$
f(x_i) = wx_i + b， with f(x_i) \approx y_i
$$

也就是说我们尽量使得f(xi)接近于yi，那么问题来了，我们如何衡量二者的差别？常用的方法是均方误差，也就是欧氏距离

$$
(w*,b*) = argmin_{(w,b)}\sum_{i=1}^m(f(x_i) - y_i)^2
$$

这里的arg min是指后面的表达式值最小时的(w, b)取值。

这就是大名鼎鼎的最小二乘法 (least square method)

### 解析解

那么上面的公式我们如何求得参数w,b呢？这里需要一些微积分（calculus)的知识。

$$
E_{w,b} = \sum_{i=1}^m (y_i - wx_i - b)^2
$$

E是关于(w,b)的凸函数(也就是boat shape的函数，只有一个最小值)，E在最小值时的(w,b)就是我们所需要求的参数值。

E对w求导：
$$
\begin{aligned}
& \frac{\partial E(w,b)}{\partial w} = \frac{\partial \sum_{i=1}^m(y_i - wx_i -b)^2}{\partial w}\\
=& \frac{\partial \sum_{i=1}^m((y_i - b)^2 + w^2x_i^2 - 2w(y_i -b)x_i)}{\partial w}\\
=& \sum_{i=1}^m(2wx_i^2 - 2(y_i -b)x_i)\\
=& 2(w\sum_{i=1}^m x_i^2 - \sum_{i=1}^m (y_i - b)x_i)
\end{aligned}
$$

E对b求导：
$$
\begin{aligned}
& \frac{\partial E(w,b)}{\partial b} = \frac{\partial \sum_{i=1}^m(y_i - wx_i -b)^2}{\partial b}\\
=& \frac{\partial \sum_{i=1}^m((y_i - wx_i)^2 + b^2 - 2b(y_i -wx_i))}{\partial b}\\
=& \sum_{i=1}^m(2b^2 - 2(y_i - wx_i))\\
=& 2(mb^2 - \sum_{i=1}^m (y_i - wx_i))
\end{aligned}
$$

最终令上面的2个导数为0，即可得到w,b的求解公式：

$$
w = \frac{\sum_{i=1}^m y_i(x_i - \overline{x})}{\sum_{i=1}^m x_i^2 - \frac{1}{m} (\sum_{i=1}^m x_i)^2}\\
~~~~~~~~~~~~~~\\
b = \frac{1}{m} \sum_{i=1}^m (y_i - wx_i)
$$

其中

$$
\overline{x} = \frac{1}{m} \sum_{i=1}^m x_i
$$

是 x 的平均值

### 逼近法

**梯度下降法**

梯度下降(Gradient Descent, GD)优化算法，是一种基于搜索的最优化方法。其作用是用来对原始模型的损失函数进行优化，以便寻找到最优的参数，使得损失函数的值最小。

> 梯度的定义
> 
> 设f是R^n中区域D上的数量场，如果f在 P0属于D 处可微，称向量
> $$
> ( \frac{\partial f}{\partial x_1}, \frac{\partial f}{\partial x_2},..., \frac{\partial f}{\partial x_n} ) |_{P_0}
> $$
> 为 f 在 P0 处的梯度，记做 ***grad*** f(P0)
> 
> 如果 f 在 P0 处可微， l0 是与 l 同向的单位向量，则
> $$
> \frac{\partial f}{\partial l} = (grad f,l_0)
> $$
> 当 ***grad*** f 与 l0 同向时，$\frac{\partial f}{\partial l}$达到最大，即 f 在 P0 处的方向导数在其梯度方向上达到最大值，此最大值即梯度的范数 ||***grad*** f||。这就是说，沿梯度方向，函数值增加最快。同样可知，方向导数的最小值在梯度的相反方向取得，此最小值即 -||***grad*** f||，从而沿梯度相反方向函数值的减少最快。
> 
> 在单变量的函数中，梯度其实就是函数的微分，代表着函数在某个给定点的切线的斜率；在多变量函数中，梯度就是一个向量，向量有方向，梯度的方向就指出了函数在给定点的上升最快的方向。
> 

简单的说，我们需要沿着梯度下降的方向找到一个收敛的参数值

要找到使损失函数最小化的参数，如果纯粹靠试错搜索，比如随机选择1000个值，依次作为某个参数的值，得到1000个损失值，选择其中那个让损失值最小的值，作为最优的参数值，那这样太笨了。我们需要更聪明的算法，从损失值出发，去更新参数，且要大幅降低计算次数。

梯度下降算法作为一个聪明很多的算法，抓住了参数与损失值之间的导数，也就是能够计算梯度（gradient），通过导数告诉我们此时此刻某参数应该朝什么方向，以怎样的速度运动，能安全高效降低损失值，朝最小损失值靠拢。

$$
repeat \ until \ converge : \\
\theta_j := \theta_j - \alpha \frac{\partial J(\theta)}{\theta_j}
$$

其中 alpha 是 learing rate， 是用来控制下降每步的距离（太小收敛会很慢，太大则可能跳过最优点）

其中 J函数是 所谓的 loss function ，同样在统计学、数学中有大量的应用。是衡量我们的预测函数f(x)精度的函数，同样我们的目标是最小化J。

$$
J(\theta) = \frac{1}{2m} \sum_{i=1}^m (h_{\theta} (x^{(i)}) - y^{(i)} )^2
$$

注意到其中的参数1/2m，这个参数是可以简化部分求导（消掉2m)。除了参数外，其它部分与解析解部分是完全相同的（事实上，解析解部分的公式也是cost function)。

这里的参数是不重要的，因为α的选择可以对冲掉。

注意上面的θ和解析解部分的w,b是一样的（只是换了下字母）。

#### 问题

从理论上，它只能保证达到局部最低点，而非全局最低点。在很多复杂函数中有很多极小值点，我们使用梯度下降法只能得到局部最优解，而不能得到全局最优解。那么对应的解决方案如下：首先随机产生多个初始参数集，即多组；然后分别对每个初始参数集使用梯度下降法，直到函数值收敛于某个值；最后从这些值中找出最小值，这个找到的最小值被当作函数的最小值。当然这种方式不一定能找到全局最优解，但是起码能找到较好的。

初始值的点，也是一个超参数

### 如何选择

两种方法（解析解和逼近法）如何选择，解析解好处是显而易见的，例如无需选择learning rate alpha，无需循环，坏处是要有复杂的矩阵运算（转置矩阵，逆矩阵等），复杂度比较高O(n^3)，相较而言，梯度下降则是O(d*n^2)

Andrew ng提到n < 10000，大致可以用解析解，n > 10000则可以用梯度下降。当然这不是必然的，我们需要理解为什么不选择解析解，主要是我们的计算机太慢（复杂度太高导致时间过长），随着计算机越来越快，这个经验值可以调整。

## 线性回归与逻辑回归

|  | 应用 | 变量的特性 | 输出的结果 | 变量的关系 |
| --- | --- | --- | --- | --- |
| 线性回归 | 回归问题 | 连续的变量 | 符合线性关系 | 直观表达变量关系 |
| 逻辑回归 | 分类问题 | 离散的变量 | 可以不符合线性关系 | 无法直观表达变量关系 |

