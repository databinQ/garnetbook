## 作用

**Focal loss**是**二分类(多分类)交叉熵**的变种, 目的是解决**分类问题中类别不平衡**造成的不同类别分类难度的差异.

其原理是对分类难度有差异的样本, 造成的损失差别对待:

- 更难分类的样本(表现为预测值与真实值差别很大)我们更关注, 应当进一步放大其产生的loss
- 而接近正确的分类虽然也会造成损失, 但这部分损失我们并不关心, 应当进一步缩小
  - 不能完全抛弃, 否则在训练过程中样本刚越过阈值, 被分类正确后, 就不会产生误差了, 影响进一步的提升, 得不到训练, 甚至在之后的训练步中被错误分类

## 形式

$$L_{fl}=\left\{\begin{aligned}&-(1-\hat{y})^{\gamma}\log \hat{y},\quad y=1\\ &-\hat{y}^{\gamma}\log (1-\hat{y}),\quad y=0\end{aligned}\right.$$

Focal loss的形式如上式. 它是按照如何解决样本不平衡分类的问题呢?

比如负样本远比正样本多的话, 模型肯定会倾向于数目多的负类(极端例子就是全部样本都判为负类), 即所有样本产生的预测$$\hat{y}$$就都会偏小, 甚至接近于0. 这时候, 负类的$$\hat{y}^{\gamma}$$普遍会很小, 而正类的$$(1-\hat{y})^{\gamma}$$就会很大, 模型因此会自动地关注正样本. 反之亦然.

这里的$$\gamma$$的作用是调节权重曲线的陡度.

正式的Focal loss如下:

$$L_{fl}=\left\{\begin{aligned}&-\alpha(1-\hat{y})^{\gamma}\log \hat{y},\,\text{当}y=1\\ &-(1-\alpha)\hat{y}^{\gamma}\log (1-\hat{y}),\,\text{当}y=0\end{aligned}\right.$$

与上面相比较, 多了一个$$\alpha$$. $$\alpha$$的作用为, 在正样本是少数样本的场景中, 由于$$(1-\hat{y})^{\gamma}$$和$$\hat{y}^{\gamma}$$的操控, 模型会更关注正样本, 但这个度可能会过, 有些极端, 因此需要用一个参数对正样本产生的loss进行降权, 将注意力效果拉回来一点.

一般取$$\alpha=0.5$$, $$\gamma=2$$即可, $$\alpha$$在不确定效果, 没有能力对其进行调参时, 最好不使用, 即用0.5即可.

上式是二分类的情况, 推广到多分类即为:

$$L_{fl}=-\alpha_{t}(1-\hat{y}_t)^{\gamma}\log \hat{y}_t$$

其中的$$\hat{y}_t$$就是经过softmax之后的对目标的预测值.

## 参考资料

以上的内容即分析基本来自于下面这篇文章, 这篇文章还有一个很好的点在于作者一步一步给出了在不平衡样本分类的需求下, 如何从普通的二分类交叉熵损失演变到focal loss, 更有助于理解focal loss的原理.

- [从loss的硬截断、软化到focal loss](https://kexue.fm/archives/4733)
