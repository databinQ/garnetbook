## 引入

也是一种Metric-based模型. 孪生模型(Siamese Neural Network)在训练和测试的时候使用的样本是以**对pair**的形式, 判断两张图片, 两句短文本等等是不是属于同一样本. 使用pair这种形式天生的劣势, 在训练和测试时, 如何合适的构造样本都是需要一定的策略的. 例如在训练时相同分类和不同分类的样本比例应为1:1等等.

而**Matching Network**使用了不同的逻辑, 它的训练和测试样本不再是一对对图片, 文本, 而是如同普通模型一样使用单个样本. 当然在One-shot任务中, 还是要分为support set和batch set的, 只不过set中的每个样本不再是一对图片, 而是一张图片.

![](img/v2-d96878965905a5de3b4a91a5710f59e7_hd.jpg)

Matching Network是一个端到端的模型. 它的结构设计处于以下考虑:

- 在判断新图片的分类时(上图右侧单图), 能够充分使用support set(上图左侧四图)中的信息. 具体来说是将新图的类别结果来源于support set中的样本标签. 这在Siamese Neural Network中是不存在

- 新图的标签预测类似于**Nearest Neighbors**或**KDE**机制, 比较与support set中样本的近似程度, 通过一定的策略/规则(如相似度加权)从supprot set中的标签, 得到预测标签. 这种类似于Nearest Neighbors或KDE的方法是**non-parametric**的

- 对于新的之前从未观测到的类别, 可以直接使用模型在这些类别上进行预测得到高质量的结果, 而且不用对训练好的模型做任何操作

## 模型解释

### 基本结构

先从模型的最后部分, 如何得到预测样本的类别来看.

对于包含$$k$$个样本的support set, 将一个样本的(image, labal)对记为$$S=\left\{\left(x_{i}, y_{i}\right)\right\}_{i=1}^{k}$$. 对于新的预测样本$$\hat{x}$$, 希望得到高质量的预测$$\hat{y}$$.

训练好的模型既是一个映射函数$$c_{S}(\hat{x})$$, 可以看到样本的预测结果是和support set $$S$$紧密相关的, 因此可以将模型记为$$P(\hat{y} | \hat{x}, S)$$. 因此test样本的预测类别为:

$$\arg \max _{y} P(y | \hat{x}, S)$$

对于test样本, 使用**attention**思想去计算它的预测类别$$\hat{y}$$:

$$\hat{y}=\sum_{i=1}^{k} a\left(\hat{x}, x_{i}\right) y_{i}$$

测试样本与support set中每个样本的注意力计算方法如下, 其中$$c$$为余弦距离, 而$$f$$和$$g$$分别是对test和support表征的函数, 简单来说可以是一些图像或者文本的表征神经网络, 输出表征向量:

$$a\left(\hat{x}, x_{i}\right)=e^{c\left(f(\hat{x}), g\left(x_{i}\right)\right)} / \sum_{j=1}^{k} e^{c\left(f(\hat{x}), g\left(x_{j}\right)\right)}$$

这种机制在文中被称为使用了`attention mechanism`, 实质上是将support set中的每个样本的表征向量依次与待预测样本的表征向量点积, 然后使用softmax函数进行**归一化**, 使用这个归一化的权重值, 对support set中所有样本的标签进行加权, 得到预测样本在support set包含的所有类别上的概率值.

这里求取test样本的类别使用了`attention mechanism`完成了类似于**KDE**的效果. 这种方法充分里使用support set中的信息. 相比而言, 以下常用的方法都有着明显的缺点:

- 最邻近: 也是依次计算test和support set中每个样本的相似度, 然后选择最相似的样本, 以这个样本的类别作为test样本的预测类别
    - 最终只使用了support set中一个样本的信息, 丢弃了大量信息
    - 这种方法类似于**kNN**(k Nearest Neighbors)机制

### Full Context Embeddings

这是论文中比较重要的一部分, 但在实际使用中可以选择性使用. 这一章节的目的是使用不同的方法, 分别对test和support set中的样本进行更好的表征, 充分利用support set中的信息. 当然如果不使用本章节的结构, 直接使用神经网络表征器(test, support set共享一套参数)也是可以的.

在目前为止的框架中, 我们使用$$f$$和$$g$$对test和support set中的样本进行embedding, 然后使用`attend`, `point`, `knn`等形式得到对test的类别预测. 这样的形式有两个不足:

- support set中的每个样本$$x_i$$是相互独立的, 得到embedding向量$$g(x_i)$$的过程也是相互独立的, 互相之间不影响
- test样本$$\hat{x}$$也是用过$$f(\hat{x})$$来得到embedding向量的, 而且现在的$$f$$和$$g$$使用的是同样的抽取器. 但由于$$\hat{x}$$的类别是由support set中的样本决定的, 那么$$\hat{x}$$的embedding向量$$f(\hat{x})$$的抽取, support set也应当参与进来

因此使用**Full Context Embeddings**这种思想的引领下, 希望对support set中的每个样本$$x_i$$以及test样本$$\hat{x}$$的样本的embedding, 都应该受到整个support set $$S$$的影响.

分别来说.

#### Full Context Embeddings $$g$$

对于support set中的样本$$x_i$$, 它的embedding向量就由$$g(x_i)$$变为了$$g(x_i, S)$$. 至于为什么这么做, 作者是这样解释的:

> This could be useful when some element $$x_j$$ is very close to $$x_i$$, in which case it may be beneficial to change the function with which we embed $$x_i$$.

如何做到整个support set $$S$$去影响样本$$x_i$$的表征过程呢? 论文中使用了**bi-LSTM**去对$$x_i$$进行编码. 使用这种方法, 就需要把support set中的所有样本考虑成一个序列, 一个一个进行输入, 依次得到每个样本的$$x_i$$的表征向量.

这里就产生了一个新的问题, 原来support set中无序的样本, 如何合理的进行排序. 文章参考了[Order Matters: Sequence to sequence for sets](https://arxiv.org/abs/1511.06391)这篇论文中的方法.

具体来说, 将原来的表征器得到表征向量(样本之间相互独立)重新记为$$g^{\prime}\left(x_{i}\right)$$, 样本新的表征$$g(x_i, S)$$为:

$$g\left(x_{i}, S\right)=\vec{h}_{i}+\overleftarrow{h}_{i}+g^{\prime}\left(x_{i}\right)$$

即原始表征向量$$g^{\prime}\left(x_{i}\right)$$和bi-LSTM对应输入位置的隐向量的加和:

$$\begin{aligned}
\vec{h}_{i}, \vec{c}_{i} &=\operatorname{LSTM}\left(g^{\prime}\left(x_{i}\right), \vec{h}_{i-1}, \vec{c}_{i-1}\right) \\ \overleftarrow{h}_{i}, \overleftarrow{c}_{i} &=\operatorname{LSTM}\left(g^{\prime}\left(x_{i}\right), \overleftarrow{h}_{i+1}, \overline{c}_{i+1}\right)
\end{aligned}$$

本质上相当于使用bi-LSTM结构对support set中的样本序列进行表征, 但添加了一个**skip connection**结构, 即最后加上的$$g^{\prime}\left(x_{i}\right)$$, 加快了训练的速度和鲁棒性.

#### Full Context Embeddings $$f$$

对于test样本$$\hat{x}$$, 这里也使用一种带**attention**的**LSMT**结构进行表征, 注意这里的LSTM与上面对support set中样本表征使用的是不同的网络(单向, 双向就能体现出来), 由于这里的表征也使用到了整个support set, 因此将新的表征向量记为:

$$f(\hat{x}, S)=\operatorname{attL} \operatorname{STM}\left(f^{\prime}(\hat{x}), g(S), K\right)$$

其中$$f^{\prime}(\hat{x})$$是原始的表征器对test样本的表征向量. $$K$$是固定的数值, 代表这里LSTM推进的步数. $$g(S)$$是一个集合, 包含了support set中每个样本的表征向量(full content embeddings得到的).

具体来说, 在执行到第$$k$$步时, 使用如下的方式得到这一步的隐向量:

$$\hat{h}_{k}, c_{k}=\operatorname{LSTM}\left(f^{\prime}(\hat{x}),\left[h_{k-1}, r_{k-1}\right], c_{k-1}\right)$$

需要注意:

- 无论在哪一步, 输入都是固定的$$f^{\prime}(\hat{x})$$, 即原始的test样本的表征向量
- 这里使用的LSTM中的隐向量是$$h_{k-1}$$和$$r_{k-1}$$拼接得到的. 因此长度是输入表征向量的两倍, 对输出的维度没有影响

得到这一步LSTM隐向量之后, 这一步的最终输出如下表示:

$$h_{k}=\hat{h}_{k}+f^{\prime}(\hat{x})$$