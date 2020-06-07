# 错误表现

运行以下代码以训练模型:

```python
train_model.fit_generator(
    train_generator.fit_iter(),
    steps_per_epoch=len(train_generator),
    epochs=20,
    callbacks=[evaluator]
)
```

遇到错误:

```python
Epoch 1/25
Traceback (most recent call last):
  File "binary_classification.py", line 59, in <module>
    history=model.fit(X, y,batch_size=10, epochs=25,validation_split=0.7)
  File "/home/isabella/.local/lib/python3.6/site-packages/keras/engine/training.py",
line 1039, in fit
    validation_steps=validation_steps)
  File "/home/isabella/.local/lib/python3.6/site-packages/keras/engine/training_arrays.py",
line 217, in fit_loop
    callbacks.on_epoch_end(epoch, epoch_logs)
  File "/home/isabella/.local/lib/python3.6/site-packages/keras/callbacks.py",
line 79, in on_epoch_end
    callback.on_epoch_end(epoch, logs)
  File "/home/isabella/.local/lib/python3.6/site-packages/keras/callbacks.py",
line 338, in on_epoch_end
    self.progbar.update(self.seen, self.log_values)
AttributeError: 'ProgbarLogger' object has no attribute 'log_values'
```

# 错误原因

一般出现`AttributeError: 'ProgbarLogger' object has no attribute 'log_values'`这种错误, 是由于代码错误, 导致**训练集**为空, 或者训练集的大小不到一个batch size. 因此在keras**更新训练的进度条**时, 发生错误.

具体来说, 可能是喂给模型的训练集是空的; 或者在使用`fit_generator`方法时, 指定**steps_per_epoch**参数为0也会导致该错误.

# 解决方法

- 检查代码中训练集生成部分, 以及`steps_per_epoch`参数是否为0.
- 如果训练集生成没有错误, 且`steps_per_epoch`参数没有问题, 可以设置`verbose=0`, 不使用进度条, 解决此问题

# 参考资料

- [github](https://github.com/keras-team/keras/issues/3657)
- [stackoverflow](https://stackoverflow.com/questions/57857683/attributeerror-progbarlogger-object-has-no-attribute-log-values)
