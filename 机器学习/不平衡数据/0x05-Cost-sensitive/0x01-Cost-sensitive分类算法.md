## Cost-sensitive

Cost-sensitive分类面向的是类型比例不平衡的数据集上的分类问题, 目的是使分类器的分类结果对应的cost值最低. Cost-sensitive方法通过引入现实世界中实际存在的**不同误判结果间代价的不平衡性**, 使得原有的机器学习算法能够在一定程度上适应现实世界应用的需要.

Cost-sensitive分类具体的技术手段分为两大类:

- rescaling
- reweighted

## Cost-matrix

在传统的分类问题中, FP和FN的代价是等同的, 这与现实生活中的很多场景是不相符合的. 例如在银行贷款是否通过的分类问题中, FN的代价是远远高于FP的. 因此, 区别误判代价之间不平衡问题的最简单的手段就是引入误判代价权重, 从而引出cost matrix. 以二分类问题为例, cost matrix为:

- |actual positive | actual negative
-|-|-
predict positive|$$c_{00}$$|$$c_{01}$$
predict negative|$$c_{10}$$|$$c_{11}$$

还是以银行贷款审批为例, 此时的实际损失就为$$|\mathrm{FP}| * \mathrm{c}_{10}+|\mathrm{FN}| * \mathrm{c}_{01}$$. 我们可以令$$c_{10}=1$$, $$c_{01}=10$$, 从而体现出误判的不平衡, 更贴合实际.

因此cost-sensitive分类算法需要考虑两个问题:

- **分类算法不仅仅能够较为准确地对数据集进行分类**
- **尽可能的控制由分类错误造成的损失**

通常情况下, 对于一个$$N$$分类问题, cost matrix是一个$$N\times{N}$$的矩阵, 将这个矩阵记为$$C$$, 其中的条目$$c(i,j)$$表示分类算法将实际属于$$j$$类的样本分类为$$i$$类所造成的的cost.

显示中cost很难准确获取, cost matrix往往是由数据集提供者通过自己 的经验给出$$c(i,j)$$的值.

## Cost-sensitive分类的目标

利用cost matrix, 可以将**cost-sensitive分类问题**转换为一个**优化问题**. 对于分类算法$$A$$, 为样本实例$$\mathbf{x}$$寻找一个类别$$i$$, 使得下式达到最小值:

$$L(\mathbf{x},i)=\sum\limits_{j}P(j|\mathbf{x})c(i,j)$$

$$P(j|\mathbf{x})$$即是在算法$$A$$中, 样本$$\mathbf{x}$$属于类别$$i$$的概率. $$L(\mathbf{x},i)$$是算法给出的所有可能类别, 以预测概率值为权重的cost值之和.

在没有引入cost时, 我们关注如何最大化真实类别对应的$$P(j|\mathbf{x})$$概率, 而现在关注最小化总的cost值. 因此Cost-sensitive分类在追寻最优cost的前提下, 可能会放弃可能性最大的分类结果.

简单起见, 考虑二分类的情况. Cost-sensitive分类算法, 将一个样本分为正样本的条件是, 造成的**分类成正样本造成的预期cost比分成负样本造成的cost小**, 公式表示如下:

$$P(j=0|\mathbf{x})c_{10}+P(j=1|\mathbf{x})c_{11} \le P(j=0|\mathbf{x})c_{00}+P(j=1|\mathbf{x})c_{01}$$

再令$$p=P(j=1|\mathbf{x})$$, 上式可写成:

$$(1-p)c_{10}+pc_{11} \le (1-p)c_{00}+pc_{01}$$

上式即算法的决策依据, 产生了决策平面.

## Cost-sensitive分类算法的现有方法

Cost-sensitive分类算法又分为**二分类**和**多分类**两大类. 而由于常用的各种分类器都是非Cost-sensitive分类算法, 如何对现有的分类器进行改造, 使其成为Cost-sensitive分类算法?

Cost-sensitive分类算法的目标是为了获得尽可能小的$$L(\mathbf{x},i)$$值. 因此为了达到上面的目的, 需要调整原有分类算法的$$P$$值, 也就是调整模型本身. 从而通过增加cost的方法, 直接影响到模型的训练, 使得模型本身发生了变化(如内部的参数).

目前有两种主要的方法:

- 通过对**训练数据集的预处理**以提高分类算法对于某些分类结果的敏感性, 这种方法称为**rescaling**
  - 多是一些**Cost-based采样**方法
- 通过以cost为基准, 修改不同分类在算法中的class membership probability, 这种方法称为**reweighted**

## 参考资料

- [Cost-sensitive 分类算法——综述与实验](http://lamda.nju.edu.cn/huangsj/dm11/files/qiny.pdf)
