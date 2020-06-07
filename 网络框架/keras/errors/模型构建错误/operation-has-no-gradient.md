# 错误表现

在使用自定义的层模型搭建完毕之后, 运行`fit`进行训练时, 报如下的错误:

```python
ValueError: An operation has `None` for gradient. Please make sure that all of your ops have a gradient defined (i.e. are differentiable). Common ops without gradient: K.argmax, K.round, K.eval.
```

# 错误原因

在自定义层中的`build`方法里, 定义了一些变量, 但是如果有变量在`call`方法中没有被使用, 就会报上面的错误.

# 解决方法

检查自定义层的定义.

# 参考资料

- [github issues](https://github.com/keras-team/keras/issues/12521#issuecomment-496743146)
- [CSDN](https://blog.csdn.net/weixin_38336626/article/details/105408627)
