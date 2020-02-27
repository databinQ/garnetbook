GELU(Gaussian Error Linear Unit), 高斯误差线性单元.

神经网络中的一些结构, 如ReLU, Dropout等结构, 都希望将网络中不重要的激活信息置为0. 以ReLU为例, 可以将函数看作在输入上乘了一个mask function, 这个函数在$$x \ge 0$$时为1, $$x \lt 0$$时为0. 同时注意到这种mask策略是与输入相关的.

GELU激活函数延续这种思想, 这里的mask function变成了$$m \sim \text { Bernoulli }(\Phi(x))$$, 其中$$\Phi(x)=P(X \leq x), X \sim \mathcal{N}(0,1)$$, $$\Phi(x)$$是标准正态分布的累积分布函数(CDF), 保持了mask与输入的相关性.

为什么是标准正态分布, 因为如今网络中广泛使用Batch Normalization结构,, 激活函数的输入基本是符合这种分布的, 使用GELU激活函数也要求分布具有这个特性.

使用$$m \sim \text { Bernoulli }(\Phi(x))$$, 输入会随着值的减小, 被drop的概率增大. 这样最终得到的激活函数是由输入$$x$$决定, 但不再有具体的参数, 即与ReLu相比是**软依赖**的.

将这个mask作用于输入, 得到:

$$x \Phi(x) = \Phi(x) \times I x+(1-\Phi(x)) \times 0 x$$

因此, 激活函数相当于按照**当前的输入有多大概率大于其他输入**对应的概率, 对输入$$x$$进行缩放. GELU激活函数最终定义为:

$$\operatorname{GELU}(x)=x P(X \leq x)=x \Phi(x)$$

以下两个函数都逼近与定义的GELU函数, 在实际中使用:

$$0.5 x\left(1+\tanh \left[\sqrt{2 / \pi}\left(x+0.044715 x^{3}\right)\right]\right)$$

$$x \sigma(1.702 x)$$

第二种逼近形式类似于**Swish**激活函数$$\operatorname{Swish}(x) = x \sigma(\beta x)$$

对应的函数图和导数图为:

![](img/601.jfif)

### 优缺点

**优点**

- 在NLP领域实践最佳, 在Transformer模型中表现很好, BERT, RoBERTa, ALBERT的二阶段模型中都使用了这种激活函数
- 类似于ReLU, 能够避免梯度消失的问题

**缺点**

- 计算量大
