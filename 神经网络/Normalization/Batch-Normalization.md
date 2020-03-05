# Batch Normalization

[Batch Normalization: Accelerating Deep Network Training by Reducing Internal Covariate Shift](https://arxiv.org/abs/1502.03167)论文中提出了**Batch Normalization**这种结构. 从名字中可以看出, Batch Normalization是设计用来消除Internal Covariate Shift问题的, 这种问题出现的原因, 表现和影响在[Normalization综述](Normalization综述.md)已经详细说明过了.

对于Mini-Batch SGD优化算法, 一次迭代中包含若干个样本. 不同迭代轮次使用的不同batch之间, 样本之间的分布差别可能会很大, 因此无论输入层还是隐层的分布都是不稳定的, 出现ICS问题.

Batch Normalization的主要思想, 是对于每个**神经元**, 数据在进入**激活函数之前**, 计算出每个神经元在这个batch中的均值和方差, 然后通过偏移, 缩放, 以及再偏移, 再缩放, 使得这个神经元保持相对稳定的输入分布.

## 训练过程

Batch Normalization针对单个神经元进行. 假设一个batch中有$$m$$个样本, 对于FNN网络, 在一个batch中进行如下步骤:

$$\mu=\frac{1}{m} \sum_{i=1}^{m} z^{(i)}$$

$$\sigma^{2}=\frac{1}{m} \sum_{i=1}^{m}\left(z^{(i)}-\mu\right)^{2}$$

$$z_{\text {Norm }}^{(i)}=\frac{z^{(i)}-\mu}{\sqrt{\sigma^{2}+\varepsilon}}$$

$$\tilde{z}^{(i)}=\gamma z_{\text {Nom }}^{(i)}+\beta$$

经过这几部处理后, 得到了最终输入到激活函数中的数据, 具有分布的稳定性.

## 推理过程

在推理阶段, 一般输入只有一个实例, 而不是与输入相同大小的batch size. 这时就不能再使用输入数据计算均值和方差了.

这就需要使用训练阶段的统计数据来作用了. 具体来说, 在推理阶段, 直接使用训练阶段所有的样本实例, 计算**全局统计量**, 来代替一个batch中的统计量. 这样更为准确. 如何获得这个全局统计量呢?

对于训练阶段的每次迭代, 都会得到这次迭代batch中$$m$$个样本的均值和方差, 使用这里的均值和方差的期望, 即计算所有batch均值的平均值得到全局均值, 计算所有batch方差的无偏估计得到全局方差:

$$
\begin{aligned}
\mathrm{E}[x] & \leftarrow \mathrm{E}_{\mathcal{B}}\left[\mu_{\mathcal{B}}\right] \\
\operatorname{Var}[x] & \leftarrow \frac{m}{m-1} \mathrm{E}_{\mathcal{B}}\left[\sigma_{\mathcal{B}}^{2}\right]
\end{aligned}
$$

然后每个神经元都有训练好的再偏移参数$$\beta$$和再缩放参数$$\gamma$$, 所以推理阶段的值由下式得到:

$$y=\frac{\gamma}{\sqrt{\operatorname{var}[x]+\epsilon}} \cdot x+\left(\beta-\frac{\gamma \mathrm{E}[x]}{\sqrt{\operatorname{var}[x]+\epsilon}}\right)$$

上式和$$y^{(k)}=\gamma^{(k)} \widehat{x}^{(k)}+\beta^{(k)}$$式是等价的, 之所以把$$\widehat{x}$$拆开是因为$$\frac{\gamma}{\sqrt{\operatorname{var}[x]+\epsilon}}$$和$$\left(\beta-\frac{\gamma \mathrm{E}[x]}{\sqrt{\operatorname{var}[x]+\epsilon}}\right)$$对于每个神经元都是可以提前计算好, 存储起来的, 在推理时直接使用, 加速了推理的效率.

## BN与CNN

在CNN中使用Batch Normalization, 秉承了CNN的核心思想, **权值共享**, 对于每个通道, 使用同样的参数, 类似于将一个通道作为一个神经元来处理.

假设卷积后, 进入激活函数之前, 此时的**feature map**的大小为$$(N, C, H, W)$$, 其中$$N$$是batch size, $$C$$为通道数量, $$H$$和$$W$$分别为高和宽. 对于每个通道, 我们计算得到一个均值和方差**标量**, 并设置一个可训练的再偏移参数和再缩放参数**标量**, 因此卷积层最后得到的均值, 方差等向量的长度为该层通道的数量.

单个通道的均值和方差计算公式如下, 都是标量:

$$\mu_{c}(x)=\frac{1}{N H W} \sum_{n=1}^{N} \sum_{h=1}^{H} \sum_{w=1}^{W} x_{n c h w}$$

$$\sigma_{c}(x)=\sqrt{\frac{1}{N H W} \sum_{n=1}^{N} \sum_{h=1}^{H} \sum_{w=1}^{W}\left(x_{n c h w}-\mu_{c}(x)\right)^{2}+\varepsilon}$$

可以看到, 是在$$N \times H \times W$$上进行的计算.

## BN的使用位置

**全连接层或卷积操作之后, 激活函数之前.**

## BN与RNN



# 参考资料

- [Batch Normalization导读](https://zhuanlan.zhihu.com/p/38176412)
