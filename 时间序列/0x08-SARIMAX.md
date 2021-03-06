## SARIMAX

**SARIMA**模型是综合了:

- ARIMA模型
- 季节性, 季节ARIMA模型
- 外因变量(exogenous)

其中的**外因变量**在`ARIMAX`和`SARIMA`模型中对应那个`X`, 是对时间变量产生影响的, 除开时间序列本身的变量. 这个变量其实也是一个序列, 而且与时间序列中在每个时间点上都是一一对应的关系.

如何使用这个外因变量呢. 通常是在时间序列中, 在SARIMAX模型中指定`exog`为这个外因变量.

需要注意的是, 在对未来进行预测的时候, 要提供这个外因变量的未来值. 因此要提前通过拟合的方法, 生产**未来的外因变量**, 然后才能对时间序列的未来做预测.

