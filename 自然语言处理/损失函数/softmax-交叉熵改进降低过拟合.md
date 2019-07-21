多分类问题中常常使用softmax函数输出类别概率, 然后配合交叉熵计算得到最终loss. 这种方案虽然常用, 但有不少缺陷, 例如:

- 分类太自信. 表现为分类的结果概率往往非1即0的, 即使输入噪音, 得到的也是概率非常高的分类, 这往往代表着**过拟合**严重

对这个问题进行具体的分析, 并提出改进softmax+交叉熵的方法.

## 问题原因

整个模型, 要拟合的就是一个One hot分布(label对应的), 而交叉熵公式为:

$$S(q|p)=-\sum_i q_i \log p_i$$

其中, $$q_i$$是真实的分布, 值为0或1; $$p_i$$为第$$i$$类的预测概率分布. 例如一个三分类问题, 当前样本对应的模型预测值为$$[z_1,z_2,z_3]$$, 且样本真实标签对应的分布为$$[1,0,0]$$, 则这个样本对应的损失为:

$$loss = -\log \Big(e^{z_1}/Z\Big),\, Z=e^{z_1}+e^{z_2}+e^{z_3}$$

而模型有一个偷懒减少loss的方法: 通过**增大模型中的参数**, 使得$$z_1,z_2,z_3$$都增加足够大(同scale)的比例(等价于$$[z_1,z_2,z_3]$$向量的模长), 那么在**指数函数**的背景下, $$e^{z_1}/Z$$就能够足够接近1, 因此loss就会接近0, 从而达到了目的, 但是没有达到训练的目的.

因此这种方法训练出来的分类模型, 因为参数大, 往往非常自信.

## 调整

为了减缓这种情况, 一种方案就是不去单纯的拟合One hot分布, 而是分出一小部分去拟合**均匀分布**, 对应的新loss为:

$$loss = -(1-\varepsilon)\log \Big(e^{z_1}/Z\Big)-\varepsilon\sum_{i=1}^n \frac{1}{3}\log \Big(e^{z_i}/Z\Big),\, Z=e^{z_1}+e^{z_2}+e^{z_3}$$

拟合均匀分布的loss只占$$\varepsilon$$这么一小部分. 这样, 盲目增大参数, 使得$$e^{z_1}/Z$$接近于1, 这种情况就不再是优化loss时的最优解了, 因此缓解了softmax函数带来的过分自信的情况, 同时缓解了过拟合的现象.

## 代码

对应的在`keras`框架中的代码为:

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

## 参考资料

- [Keras中自定义复杂的loss函数](https://kexue.fm/archives/4493)
