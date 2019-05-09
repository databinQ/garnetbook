首先声明变量符号. 以下所有表示都是针对单个样本, 多样本(batch)只需将向量表示扩展为矩阵表示即可.

对于多分类问题, 假设有$$m$$个分类, 则真实的标签向量是一个One-Hot向量$$\mathbf{y}\in \mathbb{R}^m$$, 其中$$j$$是真实类别对应的索引, 则对于$$i=1,2,\cdots,m$$有:

$$y_i = \left\{
\begin{array}{ll}
1, & i = j \\
0, & i \ne j 
\end{array} 
\right.$$

另外, 将softmax函数的入参记为$$\mathbf{z} \in \mathbb{R}^m$$, 表示经过前面结构转换后的输出值, softmax函数的作用就是将这个值归一化后输出, 因此每个类别的对应的这个输出值, 可以作为预测概率, 所有类别的输出之和为1. 将softmax函数的输出向量记为$$\mathbf{a} \in \mathbb{R}^m$$:

$$\mathbf{a}=softmax(\mathbf{z})$$

对于其中的每个元素$$a_i, i=1,2,\cdots,m$$, 就是最后的输出值, 有:

$$
\begin{aligned}
a_i = \frac{e^{z_i}}{\sum\limits_{k=1}^{m}e^{z_k}}
\end{aligned}
$$

其中连加符号的索引记号用$$k$$只是因为$$i$$被占用了, 在这里特指第$$i$$个元素.

对于单个样本, 其**交叉熵**以向量的形式表示为:

$$l=-\mathbf{y}\ln \mathbf{a}=-\sum\limits_{i} y_i\ln a_i=-\ln a_j$$

其中只有$$y_j=1$$, 其余的$$y_i=0$$. softmax和交叉熵经常一起出现, 因此这种组合的导数在后向传播中经常被使用的, 需要理解深刻.

这里说的导数, 即偏函数$$\frac{\partial l}{\partial \mathbf{z}}$$, 它本身也是一个长度为$$m$$的向量, 每个元素是最后的损失对于对应$$z_i$$的导数. 将$$l$$对每个元素$$z_i$$求导, 需要分为两种情况.

**情况一**, $$i=j$$

这一项比较特殊, 因为它对应的是真实类别$$j$$, 推导如下:

$$
\begin{aligned}
\frac{\partial l}{\partial z_i} &= -\frac{\partial}{\partial z_i}\ln a_j = -\frac{\partial}{\partial z_i}\ln a_i = -\frac{\partial}{\partial z_i}\ln \frac{e^{z_i}}{\sum\limits_{k=1}^{m}e^{z_k}} \\
&= -\frac{1}{\frac{e^{z_i}}{\sum\limits_{k=1}^{m}e^{z_k}}}\frac{e^{z_i} \cdot \sum\limits_{k=1}^{m}e^{z_k} - e^{2z_i}}{(\sum\limits_{k=1}^{m}e^{z_k})^2} \\
&= -\frac{\sum\limits_{k=1}^{m}e^{z_k} - e^{z_i}}{\sum\limits_{k=1}^{m}e^{z_k}} \\
&= -(1 - \frac{e^{z_i}}{\sum\limits_{k=1}^{m}e^{z_k}}) \\
&= a_i - 1 \\
&= a_i - y_i
\end{aligned}
$$

**情况二**, $$i \ne j$$

$$
\begin{aligned}
\frac{\partial l}{\partial z_i} &= -\frac{\partial}{\partial z_i}\ln a_j = -\frac{\partial}{\partial z_i}\ln \frac{e^{z_j}}{\sum\limits_{k=1}^{m}e^{z_k}} \\
&= -\frac{1}{\frac{e^{z_j}}{\sum\limits_{k=1}^{m}e^{z_k}}} \frac{-e^{z_i}e^{z_j}}{(\sum\limits_{k=1}^{m}e^{z_k})^2} \\
&= \frac{e^{z_i}}{\sum\limits_{k=1}^{m}e^{z_k}} \\
&= a_i \\
&= a_i - y_i
\end{aligned}
$$

此时的$$y_i=0$$.

因此综上推导过程, softmax函数结合交叉熵损失:

$$
\begin{aligned}
\frac{\partial l}{\partial z_i} = a_i - y_i
\end{aligned}
$$

用向量表示为:

$$
\begin{aligned}
\frac{\partial l}{\partial \mathbf{z}} = \mathbf{a} - \mathbf{y}
\end{aligned}
$$
