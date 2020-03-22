ArcFace, CosFace, 和SphereFace三者都是Softmax Loss的改进. 最初都是在人脸识别领域使用. Softmax Loss有着如下的缺点:

- 使用分类的方式进行人脸识别都会遇到训练和测试不一致的问题. 例如使用10万张人脸数据进行训练, 但是在应用时可能对几百个, 或几百万人脸进行识别. 而使用Softmax Loss训练得到的feature embedding, 在test阶段, 同一类中的样本不能很好的聚集在一起, 即类别之前的区分程度不足
- Softmax Loss在输出层引入的参数$$W$$是与分类类别线性相关的, 对于人脸识别这种类别数量很多的情景, 会导致输出层的linear transformation matrix中的参数数量过多

因此, ArcFace, CosFace, 和SphereFace都是为了提成对不同类别的区分能力诞生的.

# Softmax Loss

首先来看最常用的Softmax Loss:

$$L_{1}=-\frac{1}{N} \sum_{i=1}^{N} \log \frac{e^{W_{y_{i}}^{T} x_{i}+b_{y_{i}}}}{\sum_{j=1}^{n} e^{W_{j}^{T} x_{i}+b_{j}}}$$

第$$i$$个样本
