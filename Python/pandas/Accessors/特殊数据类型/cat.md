## 类型转换

**cat**这个访问器是对pandas中的类别类型的列进行特定的操作. 使用前, 需要将对应的列转换成类别类型.

创建时指定为`category`格式:

```python
In [1]: s = pd.Series(["a", "b", "c", "a"], dtype="category")

In [2]: s
Out[2]: 
0    a
1    b
2    c
3    a
dtype: category
Categories (3, object): [a, b, c]
```

## 操作

关于**cat**的操作详见[cat](http://pandas.pydata.org/pandas-docs/stable/reference/series.html#api-series-cat).
