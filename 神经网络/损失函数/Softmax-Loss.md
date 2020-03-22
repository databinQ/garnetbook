Softmax Loss即Softmax-交叉熵损失函数.

## Softmax-交叉熵求导

参考[Softmax-交叉熵求导](Softmax-交叉熵求导.md).

## Softmax-交叉熵缺点

### 过分自信

使用这种组合训练分类任务, 得到的分类结果几乎是非1即0的, 会造成一些问题, 表现为:

- 输出的值更多是对结果的confidence, 而不是分类概率意义
- 如果下游需要使用阈值对数值进行划分, 得到的结果往往是很差的
- 导致过拟合

这个问题产生的原因为: 如果使用的标签是one-hot标签, 网络趋向于将输入到softmax函数中的feature的**norm无限拉长**导致的.

具体来说, 对于One-Hot类型的标签, 有:

$$
\begin{aligned}
L=\operatorname{Loss}(a, y)=-\sum_{j} y_{j} \ln a_{j}=-\ln\frac{e^{z_i}}{Z}
\end{aligned}
$$

其中$$i$$为样本的正确标签, $$y_{i} = 1$$, $$y_{j\ne i} = 0$$. 假设任务是三分类任务, 且样本的标签为$$[1,0,0]$$, 则有:

$$\operatorname{Loss} = -\log \Big(e^{z_1}/Z\Big)$$

$$Z=e^{z_1}+e^{z_2}+e^{z_3}$$

预测的结果只要是$$z_1$$是$$[z_1,z_2,z_3]$$中的最大值, 就能够得到正确的分类. 而为了减小损失, $$e^{z_1} / Z$$的值肯定是越大越好. 这就是One-Hot类型的标签下Softmax-交叉熵过分自信的原因, 损失的形式决定了输出的极端.

为了增大$$e^{z_1}$$在$$Z$$中的比重, 一个简单的方法就是**同比例的增大$$z_1,z_2,z_3$$的值**, 相当于增加$$[z_1,z_2,z_3]$$向量的模长, 即上文中所说的将输入到softmax函数中的feature的**norm无限拉长**.

举个例子, 假如$$[z_1,z_2,z_3]=[1.5,1.2,1.0]$$, 对应的$$[a_1,a_2,a_3]=[0.42601251, 0.31559783, 0.25838965]$$. 如果将模长放大5倍, 即$$[z_1,z_2,z_3]=[7.5,6.0,5.0]$$, 此时对应的$$[a_1,a_2,a_3]=[0.76615721, 0.17095278, 0.06289001]$$. 可以看到此时的输出相对极端了.

而**同比例的增大$$z_1,z_2,z_3$$的值**是可以很简单的做到的, 就是通过**增大模型参数**, 因为$$z=Wx$$. 这就是softmax过于自信的来源, 只要增大参数的值, 就可以盲目地增大`logit`的模长, 增大$$e^{z_1} / Z$$的值, 使得loss降低. 这个训练过程是没有多少代价的.

这样训练得到的模型, 在每个标签的输出上都更像是**0-1阶跃函数**. 二分类softmax等价于sigmoid, 这样训练得到的函数$$sigmoid(x)=sigmoid(wx)$$, 会越来越接近**0-1阶跃函数**, 输出基本在0,1附近, 中间态几乎没有.

**参考资料**:

- [Keras中自定义复杂的loss函数](https://kexue.fm/archives/4493)
- [为什么softmax很少会出现[0.5, 0.5]?](https://www.zhihu.com/question/362870151)

#### 解决方法

减缓这种情况的一个思路, 就是不再去拟合One-hot分布.

- 一种思路是使用**soft label**, 例如二分类就使用$$[0.5, 0.5]$$这种label
- **noise label**也是一种思路, 对于一些的样本, 在训练过程的不同batch中, 使用不同的label

另外还有一种方法是在softmax loss的基础上, 不再去单纯的拟合One-hot分布, 而是分出一小部分去拟合**均匀分布**, 对应的新loss为:

$$loss = -(1-\varepsilon)\log \Big(e^{z_1}/Z\Big)-\varepsilon\sum_{i=1}^n \frac{1}{3}\log \Big(e^{z_i}/Z\Big),\, Z=e^{z_1}+e^{z_2}+e^{z_3}$$

拟合均匀分布的loss只占$$\varepsilon$$这么一小部分. 这样, 盲目增大参数, 使得$$e^{z_1}/Z$$接近于1, 这种情况就不再是优化loss时的最优解了, 因此缓解了softmax函数带来的过分自信的情况, 同时缓解了过拟合的现象.

这个方案对应的在`keras`框架中的代码为:

```python
from keras.layers import Input,Embedding,LSTM,Dense
from keras.models import Model
from keras import backend as K

word_size = 128
nb_features = 10000
nb_classes = 10
encode_size = 64

input = Input(shape=(None,))
embedded = Embedding(nb_features,word_size)(input)
encoder = LSTM(encode_size)(embedded)
predict = Dense(nb_classes, activation='softmax')(encoder)

def mycrossentropy(y_true, y_pred, e=0.1):
    loss1 = K.categorical_crossentropy(y_true, y_pred)
    loss2 = K.categorical_crossentropy(K.ones_like(y_pred)/nb_classes, y_pred)
    return (1-e)*loss1 + e*loss2

model = Model(inputs=input, outputs=predict)
model.compile(optimizer='adam', loss=mycrossentropy)
```

这里的$$\varepsilon$$对应为写死的0.1, 最好在上面的函数外再包一层工厂函数, 将损失函数作为闭包返回.

