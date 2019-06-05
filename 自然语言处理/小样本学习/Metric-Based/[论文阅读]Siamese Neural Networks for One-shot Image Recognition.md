## 引入

Algorithms often break down when forced to make predictions about data for which little supervised information is available. 对于样本量很少的类别, 现有的分类算法在分类时, 表现很差, 我们希望在不需要扩充样本的情况下, 能够有效的将算法的能力在少数类中得到提升.

例如, 对于分类任务, 对于有些类别, 我们有些类别, 我们只观察到一个样本(训练集中), 然后就需要对一个测试样本预测它的类别. 我们将其称之为**one-shot learning**.

文章提出一种新的方法, 通过多层非线性的深度学习框架, 自动地获取输入的表征, 而且这种表征能力具有很好的泛化性, 能够在只有少量样本的类别中也有很好的表现. 然后再结合one-shot learning的特点, 将分类过程分为不同的特征, 总体获得即有鲁棒性, 又能够在同类识别上表现很好.

## 方法

这是一篇图像领域的文章, 样本因此都是图像. 自然语言领域可以类比.

我们通过metric-based的监督学习模型, 即siamese neural networks来学习图像(文本)的表征, 然后进行one-shot learning任务, 重复使用这个神经网络提取特征(而不需要重新训练).

上面所述的过程用图表示如下:

![](img/微信截图_20190605221153.png)

对于图像的表征, 采用了**large siamese convolutional neural networks**, 主要考虑到:

- 能够学习到图像的通用特征(共性特征), 从而样本数很少的类别中的样本进行预测时, 很有帮助
- 从数据源中采样得到的样本对, 用标准的优化方法就能够进行训练
- 用深度学习的方法就能够得到一个很棒的方法, 从而摆脱了对专业领域的知识的依赖

为了得到这种one-shot分类模型, 首先要获得一个神经网络模型, 能够很好地对图片对是否属于一个类别进行区分, 这就是基础的图像验证任务. 因此我们认为, 那些表现很好的用于图像验证的神经网络模型, 都能够很好的扩展到one-shot classification任务当中来.

这种verification model(图像验证模型)将两张图片是否属于同一类别用概率表示. 在训练得到这种模型后, 在做测试预测时, 对于一张新的图片, 就可以判断它是否属于一些新颖的类别了(通过组成图片对来进行), 对于这张图片组成的若干个image pair, 选取具有最高分数的pair作为最有可能的结果, 从而完成了one-shot任务.

## Siamese Networks

Siamese Networks(孪生网络)就是为了完成上述的Verification工作的. Siamese Networks是一个比较古老的模型, 在90年代就被提了出来.

Siamese Networks如同它的名字一样, 包含了一堆twin networks(但不共享参数), 分别接受不同的inputs, 并且在网络的最后使用一个**energy function**连接起来. energy function的作用是在网络的末端抽取到最终的表征后, 计算度量两边的相似性.

twin networks之间的参数虽然不是共享的, 但也是会有高度的一致性关系. 这种一致性关系表现在将两个极其相似的图片放入网络中, 两边抽取到的表征向量在向量空间中不会有很大的差异. 而且这种结构是对称的, 对于两张图片, 只要我们放入的位置相同, 末层得到的metric都会是一样的.

整体的图示如下:

![](img/微信截图_20190606072508.png)

## 模型结构

首先是多层卷积层进行表征(对于NLP问题, 也可以更换成其他如RNN, Transformer等), 在一层全连接层之后, 再接上最顶层的energy function层评价相似性.

具体来说, siamese convolutional neural network 拥有$$L$$层卷积, 每层的卷积核的通道数量为$$N_l$$. 我们使用$$\mathbf{h}_{1,l}$$表示first twin第$$l$$层输出的隐向量, 同理用$$\mathbf{h}_{2,l}$$表示second twin第$$l$$层输出的隐向量.

每个卷积层都用**ReLU**函数进行激活. 后面的全连接成或最后的metric层用**sigmoid**进行激活.

这里的卷积层使用的卷积核比较特殊, 每层使用$$N_l$$个卷积核, 不同层的卷积核有着不同的size, 但步幅(stride)都是固定为1的. 每层卷积核的数量为16的倍数. 使用ReLU激活函数, 并且每层的最后使用一个与卷积核size一直, 步幅为2的max pool. 因此第$$l$$层的第$$k$$个卷积核的输出为:

$$
\begin{array}{l}
{a_{1, m}^{(k)}=\operatorname{max-pool}\left(\max \left(0, \mathbf{W}_{l-1, l}^{(k)} \star \mathbf{h}_{1,(l-1)}+\mathbf{b}_{l}\right), 2\right)} \\
{a_{2, m}^{(k)}=\operatorname{max-pool}\left(\max \left(0, \mathbf{W}_{l-1, l}^{(k)} \star \mathbf{h}_{2,(l-1)}+\mathbf{b}_{l}\right), 2\right)}
\end{array}
$$

其中$$\mathbf{W}_{l-1, l}^{(k)}$$是一个三维的张量