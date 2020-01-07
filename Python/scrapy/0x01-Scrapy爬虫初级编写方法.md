## 起始教程

### 创建项目

使用**scrapy**编写爬虫首先需要**创建项目**. 与普通的Python项目不同, 新的scrapy项目需要通过命令行的方式:

```
scrapy startproject <项目名称>
```

这条命令将会创建一个以`<项目名称>`为名的目录, 内部结构如下:

```
<项目名称>/
    scrapy.cfg
    <项目名称>/
        __init__.py
        items.py
        pipelines.py
        settings.py
        spiders/
            __init__.py
```

这些内部文件/目录的作用是:

- `scrapy.cfg`: 项目的配置文件
- `<项目名称>/`: 内部目录, 项目的python模块, 爬虫的开发代码
- `<项目名称>/items.py`: 定义使用的Item
- `<项目名称>/pipelines.py`: 项目使用到的pipeline
- `<项目名称>/settings.py`: 设置文件
- `<项目名称>/spiders/`: 放置spider代码的目录

首先了解每个文件/模块的意义, 这些是scrapy组织内容结构和定义流程的方式.

### 定义Item

**Item**是保存**爬取到的数据**的容器. 定义的方法类似于`sqlalchemy`包所使用的ORM(对象关系映射)形式. 例如我们想定义爬取到的一条数据中, 包含网址的名字, URL, 以及对应的描述等三个字段, 定义方法如下:

```python
import scrapy

class DmozItem(scrapy.Item):
    title = scrapy.Field()
    link = scrapy.Field()
    desc = scrapy.Field()
```

即创建一个`scrapy.Item`类, 并且定义每个字段为一个`scrapy.Field`. 这是Item的定义方式, 在使用的时候, 方法与python字典的用法类似.

### 编写提取数据的Spider

上一步定义了需要爬取的数据的格式, 现在则是要编写代码来提取这种数据, 即编写一个Spider. Spider是用户编写用于从单个网站(或者一些网站)爬取数据的类. 其包含了:

- 一个用于下载的初始URL
- 如何跟进网页中的链接以及如何分析页面中的内容
- 提取生成 item 的方法

现在继承`scrapy.Spider`类, 创建一个Spider, 并且自定义以下属性:

- `name`: 用于区别不同的Spider, 该名字必须是唯一的, 不可以为不同的Spider设定相同的名字
- `start_urls`: 包含了Spider在启动时进行爬取的url**列表**. 因此, 第一个被获取到的页面将是其中之一, 后续的URL则从初始的URL获取到的数据中提取
- `parse()`: spider的一个方法
    - 该方法负责
        - 解析返回的数据(response data)
        - 提取数据(生成item)
        - 生成需要进一步处理的URL的`Request`对象
    - 在被调用时, 每个初始URL完成下载后生成的`Response`对象, 将会作为**唯一**的参数传递给该函数

以下为一个Spider例子:

```python
import scrapy

class DmozSpider(scrapy.spiders.Spider):
    name = "dmoz"
    allowed_domains = ["dmoz.org"]
    start_urls = [
        "http://www.dmoz.org/Computers/Programming/Languages/Python/Books/",
        "http://www.dmoz.org/Computers/Programming/Languages/Python/Resources/"
    ]

    def parse(self, response):
        filename = response.url.split("/")[-2]
        with open(filename, 'wb') as f:
            f.write(response.body)
```

Spider代码保存在`<项目名称>/spiders/`中, 新建一个python脚本如`dmoz_spider.py`文件.

#### 爬取过程

这样就定义好了一个非常简单的爬虫. 通过命令启动Spider, 进行爬取工作:

```python
scrapy crawl dmoz
```

这里的`dmoz`为上一步定义的`DmozSpider`类的名字(`name`). 在定义时要保证不同Spider的name是唯一的.

爬虫的原理简单来说, Scrapy为Spider的`start_urls`属性中的每个URL创建了`scrapy.Request`对象, 并将`parse`方法作为**回调函数(callback)**赋值给了每个`Request`对象. Request对象经过调度, 执行生成`scrapy.http.Response`对象, 并送回给spider的`parse`方法.

#### 选择器(Selectors)

上面的代码实际上并没有使用之前定义的`DmozItem`类, 而是简单的将网页中的内容保存在文件中. 现在我们将会从网页内容中提取数据, 保存在`DmozItem`对象中.

Scrapy使用了基于**XPath**和**CSS**表达式机制来提取网页中的数据, 这套提取数据的机制在代码中对应为**选择器(Selectors)**.

XPath表达式的使用方法参考: [XPath 教程](https://www.w3school.com.cn/xpath/index.asp)

Selector有四个基本的方法:

- `xpath()`: 传入xpath表达式, 返回该表达式所对应的所有结点的`selector list`列表
- `css()`: 传入css表达式, 返回该表达式所对应的所有结点的`selector list`列表
- `extract()`: 序列化该节点为unicode字符串并返回list
- `re()`: 根据传入的正则表达式对数据进行提取, 返回unicode字符串list列表

#### 查看网页内容

现在我们开始编写一个Selector. 首先需要知道爬取网站的组织格式, 我们提取的数据是保存在哪里的. 之前一般是通过浏览的`F12`方法去查看源码. 除此之外, scrapy提供了一种内置的`Scrapy shell`工具, 运行在IPython终端中, 供我们使用scrapy selector的方法对网页进行探索. 执行以下的命令来启动shell:

```
scrapy shell "http://www.dmoz.org/Computers/Programming/Languages/Python/Books/"
```

shell载入后, 会生成一个`response`变量, 使用`response.body`属性就能获得response的返回体, 同理`response.headers`可以看到返回头. 这是scrapy组织输出的形式.

同时, shell也初始化了变量`sel`, 这是一个加载了上面response的selector, 使用对应的方法就能进行提取:

```
In [1]: sel.xpath('//title')
Out[1]: [<Selector xpath='//title' data=u'<title>Open Directory - Computers: Progr'>]

In [2]: sel.xpath('//title').extract()
Out[2]: [u'<title>Open Directory - Computers: Programming: Languages: Python: Books</title>']

In [3]: sel.xpath('//title/text()')
Out[3]: [<Selector xpath='//title/text()' data=u'Open Directory - Computers: Programming:'>]

In [4]: sel.xpath('//title/text()').extract()
Out[4]: [u'Open Directory - Computers: Programming: Languages: Python: Books']

In [5]: sel.xpath('//title/text()').re('(\w+):')
Out[5]: [u'Computers', u'Programming', u'Languages', u'Python']
```

可以看到Selector对象的使用方法: 首先使用`xpath`或`css`方法获得命中的`Selector`列表, 再使用`extract`或`re`方法提取其中存储的数据.

因此Selector中, 既是一种数据存储形式, 包含了要提取的数据, 其子结构也对应为Selector, 也是提取各种详细数据方法的提供者.

#### 提取数据

使用Selector来提取数据, 并使用之前定义的Item对象, 对应的Spider代码变为:

```python
import scrapy

from tutorial.items import DmozItem

class DmozSpider(scrapy.Spider):
    name = "dmoz"
    allowed_domains = ["dmoz.org"]
    start_urls = [
        "http://www.dmoz.org/Computers/Programming/Languages/Python/Books/",
        "http://www.dmoz.org/Computers/Programming/Languages/Python/Resources/"
    ]

    def parse(self, response):
        for sel in response.xpath('//ul/li'):
            item = DmozItem()
            item['title'] = sel.xpath('a/text()').extract()
            item['link'] = sel.xpath('a/@href').extract()
            item['desc'] = sel.xpath('text()').extract()
            yield item
```

一般来说, Spider会将爬取到的数据以`Item`对象返回.

### 保存爬取到的数据

最简单存储爬取的数据的方式是使用`Feed exports`, 可以对爬取的数据进行序列化, 如JSON格式, 生成对应的`.json`文件.

使用的方法如下:

```
scrapy crawl dmoz -o items.json
```

需要对爬取到的item做更多更为复杂的操作, 需要编写`Item Pipeline`, 添加在`<项目名称>/pipelines.py`脚本中.
