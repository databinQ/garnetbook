# Items

爬取的主要目标就是从非结构性的数据源提取结构性数据, Scrapy提供`Item`类来满足这样的需求, 提供了类似于词典(dictionary-like)的API以及用于声明可用字段的简单语法.

## 声明Item

Item作为整体的结构化数据, 其字段(item fields)使用`Field`对象来声明. 一个Field对象指明了每个字段的元数据(metadata):

```python
import scrapy

class Product(scrapy.Item):
    name = scrapy.Field()
    price = scrapy.Field()
    stock = scrapy.Field()
    last_updated = scrapy.Field(serializer=str)
```

这种定义方法类似于`sqlalchemy`包的ORM方式, 但字段没有多种类型, 统一使用`Field`对象来定义, 更为简单. Field对象接受的值没有任何限制.

**Item**复制了标准的`dict API`, 包括初始化函数也相同, Item唯一额外添加的属性是`fields`属性, 一个包含了item所有声明的字段的字典. Item对象即字典对象, 该字典的key是字段(field)的名字, 值是声明的`Field`对象.

**Field**仅仅是内置的`dict`类的一个别名, 对象完完全全就是Python字典(dict).

## 扩展Item

可以通过继承原始的Item来扩展item, 添加更多的字段或者修改某些字段的元数据:

```python
class DiscountedProduct(Product):
    discount_percent = scrapy.Field(serializer=str)
    discount_expiration_date = scrapy.Field()
```

也可以通过使用原字段的元数据, 添加新的值或修改原来的值来扩展字段的元数据:

```python
class SpecificProduct(Product):
    name = scrapy.Field(Product.fields['name'], serializer=my_serializer)
```

这段代码在保留所有原来的元数据值的情况下添加(或者覆盖)了`name`字段的`serializer`.
