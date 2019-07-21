`keras`中的损失函数, 或者自定义损失函数直接在`model.compile`中使用的, 都是接受真实标签和类别预测概率两个变量, 计算得到的, 即**loss为预测值与真实值的某种误差函数**.

但有些任务/模型, 计算损失时比较复杂, 需要别的参数. 这里的参数指的不是超参数, 例如focal loss中的$$\gamma$$或者$$\alpha$$, 这种超参数可以使用函数工厂的方法, 返回闭包, 具体方法见[自定义损失函数](自定义损失函数.md)一节. 这里的参数, 对应的是模型中某一层的输出, 可能是计算过程中需要引入其他的参数或变量, 也可能对应多输出且损失不是简单的加和情况(有交互).

在实际中常遇到的属于这种情况的有:

- seq2seq任务, 或者序列标注任务等. NLP任务基本都需要对样本进行padding或者truncate, 对于这种序列任务, 最后的输出往往都是各个序列上损失的加和
  - 因此对于padding的位置, 计算整体损失时, 就不应当计入
  - 因此对应的损失函数需要引入**mask**变量
- 句向量或任何其他形式向量之间的匹配. 例如FAQ问题, 将问题和答案都编码成长度相同的向量, 然后计算它们的余弦相似度, 作为loss, 正确答案loss小, 错误答案反之. .因此可以使用**triplet loss**:

    $$loss = \max\Big(0, m+\cos(q,A_{\text{wrong}})-\cos(q,A_{\text{right}})\Big)$$

    只要保证正确答案的cos值比错误答案的cos高即可, 大多少不重要, 以这个思路进行训练, 就对应上面的损失函数

    可以看到这个损失函数就不是简单的是输入输出的组合事情了.

## Method 1

无论哪种方法都是单独地构造一个损失层, 区别在于:

- 如何将损失层融入到模型中
- 如何组织样本喂给模型进行fit

第一种方法以上面的FAQ为例, 参考自[Keras中自定义复杂的loss函数](https://kexue.fm/archives/4493#%E5%B9%B6%E4%B8%8D%E4%BB%85%E4%BB%85%E6%98%AF%E8%BE%93%E5%85%A5%E8%BE%93%E5%87%BA%E9%82%A3%E4%B9%88%E7%AE%80%E5%8D%95)中的第二部分.

代码如下:

```python
from keras.layers import Input,Embedding,LSTM,Dense,Lambda
from keras.layers.merge import dot
from keras.models import Model
from keras import backend as K

word_size = 128
nb_features = 10000
nb_classes = 10
encode_size = 64
margin = 0.1

embedding = Embedding(nb_features,word_size)
lstm_encoder = LSTM(encode_size)

def encode(input):
    return lstm_encoder(embedding(input))

q_input = Input(shape=(None,))
a_right = Input(shape=(None,))
a_wrong = Input(shape=(None,))
q_encoded = encode(q_input)
a_right_encoded = encode(a_right)
a_wrong_encoded = encode(a_wrong)

q_encoded = Dense(encode_size)(q_encoded) #一般的做法是，直接讲问题和答案用同样的方法encode成向量后直接匹配，但我认为这是不合理的，我认为至少经过某个变换。

right_cos = dot([q_encoded,a_right_encoded], -1, normalize=True)
wrong_cos = dot([q_encoded,a_wrong_encoded], -1, normalize=True)

loss = Lambda(lambda x: K.relu(margin+x[0]-x[1]))([wrong_cos,right_cos])

model_train = Model(inputs=[q_input,a_right,a_wrong], outputs=loss)
model_q_encoder = Model(inputs=q_input, outputs=q_encoded)
model_a_encoder = Model(inputs=a_right, outputs=a_right_encoded)

model_train.compile(optimizer='adam', loss=lambda y_true,y_pred: y_pred)
model_q_encoder.compile(optimizer='adam', loss='mse')
model_a_encoder.compile(optimizer='adam', loss='mse')

model_train.fit([q,a1,a2], y, epochs=10)
#其中q,a1,a2分别是问题、正确答案、错误答案的batch，y是任意形状为(len(q),1)的矩阵
```

首先, 我们将损失函数单独地构建为一层:

```python
loss = Lambda(lambda x: K.relu(margin+x[0]-x[1]))([wrong_cos,right_cos])
```

然后在构建Model时, 指定`output`为该`loss`:

```python
model_train = Model(inputs=[q_input,a_right,a_wrong], outputs=loss)
```

这就相当于指定模型的输出不再是概率, 而是整体的损失. 因此传入到最后计算损失的函数中, 所用的`y_pred`就是模型得到的损失了, 所以`compile`方法写为:

```python
model_train.compile(optimizer='adam', loss=lambda y_true,y_pred: y_pred)
```

这相当于自定义了一个损失函数(符合keras损失函数接受y_true和y_pred的原则), 该函数返回的就是y_pred, 而y_pred就是模型计算得到的样本损失.

由于以往对于分类任务, `output`参数指定的都是输出概率那一层, 因此在`fit`的时候对应的喂给output的就是样本真实标签. 而这里, 正确答案和错误答案都已经通过`Input`占位符的方式传入, 已经计算得到了损失, 不再需要额外的真值了, 所以代码最后一行中的$$y$$, 可以是任意符合`shape`要求的`array`, 整个训练过程都不会使用到这个值(而且在构建模型时都不用给这个值一个占位符`Input`).

## Method 2

再以seq2seq任务常见的**mask**需求为例, 介绍另一种形式, 但原理与方法一是相同的.

计算损失函数时要屏蔽占位符对应产生的损失, 因此可以写为:

```python
def _get_loss(y_pred, y_true):
    y_true = tf.cast(y_true, dtype="int32")
    loss = tf.nn.sparse_softmax_cross_entropy_with_logits(labels=y_true, logits=y_pred)
    mask = tf.cast(tf.not_equal(y_true, 0), dtype="float32")
    loss = tf.reduce_sum(loss * mask, -1) / tf.reduce_sum(mask, -1)
```

mask是在损失函数中计算的, 当然也可以在compile时直接指定`loss`为该参数. 但在seq2seq任务中, decoder的输出为预测值, 其真值也会作为**输入**, 在解码时在每个step中引导下个step的输出, 作为训练的一部分, 而这个真值是从第二个step开始到最后一个step的序列. 因此如果再单独的使用一个Input作为真实结果的占位符, 就需要对原始序列进行处理, 比较麻烦, 同一个变量两次输入也不够优雅.

使用如下的方法:

```python
loss = Lambda(lambda x: self._get_loss(*x))([output, target_decode_out])

self.model = Model([source_input, target_input], loss)
self.model.add_loss([loss])
self.model.compile(optimizer, None)

self.model.metrics_names.append("ppl")
self.model.metrics_tensors.append(Lambda(K.exp)(loss))
self.model.metrics_names.append("accuracy")
self.model.metrics_tensors.append(Lambda(lambda x: self._get_acc(x[0], x[1]))([output, target_decode_out]))

self.output_model = Model([source_input, target_input], output)
```

仍然是让loss单独作为一层, 并制定为训练模型的output, 但这里指定模型损失时不是在`compile`中使用lambda函数作为损失函数, 而是使用了模型的`add_loss`方法. 注意这里的loss要用list的形式传入.

这样如果模型的整体损失是由多部分组成, 就可以使用`add_loss`函数逐步添加, 而不用构思一个损失函数去做了.

另外需要注意, 由于已经使用了`add_loss`为模型添加了损失, 在`compile`的时候就不用再为模型指定损失了, 因此`loss`参数对应的指定为`None`.

另外的隐身为对`metrics`的灵活添加. 普通的`metrics`对应是在`compile`时以列表的形式指定, 可以指定多个. 评价函数也是如同损失函数, 接受真值和预测概率, 如果使用到模型中的层或参数, 也可以如同损失函数一样添加.

添加`metrics`方法使用到`model`的两个**属性**, `metrics_names`和`metrics_tensors`, 分别对应评价方法的名字(供显示时使用)和函数, 这个需要直接计算出评价值, 训练的时候就按`名称: 值`的形式显示.

## 参考资料

- [Keras中自定义复杂的loss函数](https://kexue.fm/archives/4493)
