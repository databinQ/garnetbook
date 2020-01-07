# Spiders

Spider类定义了如何爬取某个/某些网站, 包括了爬取的动作, 以及如何从网页的内容中提取结构化数据, Spider就是定义爬取的动作及分析某个网页的地方.

## 爬取过程

1. 以初始的URL初始化Request, 并给Request设置回调函数. 当该request下载完毕并返回时, 将生成response, 并作为参数传给该回调函数
2. 在回调函数内分析返回的内容, 返回`Item`对象或者`Request`或者一个包括二者的可迭代容器. 返回的Request对象之后会经过Scrapy处理, 下载相应的内容, 并调用设置的callback函数(相当于二次爬取)
3. 在回调函数内, 可以使用**选择器**(Selectors)或`lxml`等任何工具来分析网页内容, 并根据分析的数据生成item
4. 由spider返回的item将被存到数据库或使用`Feed exports`存入到文件中

## Spider参数

在定义Spider(`__init__`)的时候可以使用参数, 来修改其功能, 但启动这个spider是用命令来启动的, 可以使用`-a`来传递需要的参数:

```python
import scrapy

class MySpider(Spider):
    name = 'myspider'

    def __init__(self, category=None, *args, **kwargs):
        super(MySpider, self).__init__(*args, **kwargs)
        self.start_urls = ['http://www.example.com/categories/%s' % category]
        # ...
```

```
scrapy crawl myspider -a category=electronics
```

## 内置Spider

Scrapy中有多种通用的spider, 为一些常用的爬取情况提供方便. 主要有以下几种.

### scrapy.spider.Spider

Spider是最简单的spider, 每个其他的spider必须继承自该类. Spider并没有提供什么特殊的功能, 其仅仅请求给定的`start_urls`或`start_request`, 并根据返回的结果(responses)调用spider的`parse`方法.

其中比较重要的属性和方法如下.

#### name

定义spider名字的字符串. spider的名字定义了Scrapy如何定位, 并初始化spider, 所以其必须是唯一的. name是spider最重要的属性, 而且是必须的.

#### allowed_domains

可选. 包含了spider允许爬取的域名(domain)列表(list), 域名不在列表中的URL不会被跟进.

#### start_urls

URL列表. 当没有制定特定的URL时, spider将从该列表中开始进行爬取. 因此, 第一个被获取到的页面的URL将是该列表之一. 后续的URL将会从获取到的数据中提取.

#### custom_settings

配置字典, 定义爬虫的行为等. 将会覆盖项目整体配置, 如果配置键有重复的话.

#### start_requests()

该方法必须返回一个可迭代对象, 该对象包含了spider用于爬取的第一个Request. 当spider启动爬取并且未指定URL时, 该方法被调用. 当指定了URL时, `make_requests_from_url()`将被调用来创建Request对象. 该方法仅仅会被Scrapy调用一次.

该方法的默认实现是使用`start_urls`的url生成Request.

#### make_requests_from_url(url)

该方法接受一个URL并返回用于爬取的`Request`对象. 该方法在初始化request时被`start_requests()`调用，也被用于转化url为request.

默认实现中, 该方法返回的Request对象中, `parse()`作为回调函数, dont_filter参数也被设置为开启.

#### parse(response)

**当response没有指定回调函数时, 该方法是Scrapy处理下载的response的默认方法**.

parse负责处理response并返回处理的数据以及(/或)跟进的URL(Spider对其他的Request的回调函数也有相同的要求, 即request的回调函数是用来处理请求完毕后的response的).

该方法及其他的Request回调函数必须返回一个包含Request及(或)Item的可迭代的对象.

#### log(message[, level, component])

使用`scrapy.log.msg()`方法记录(log)message. log中自动带上该spider的`name`属性.

#### closed(reason)

当spider关闭时, 该函数被调用.

#### Spider样例

```python
import scrapy
from myproject.items import MyItem

class MySpider(scrapy.Spider):
    name = 'example.com'
    allowed_domains = ['example.com']
    start_urls = [
        'http://www.example.com/1.html',
        'http://www.example.com/2.html',
        'http://www.example.com/3.html',
    ]

    def parse(self, response):
        self.log('A response from %s just arrived!' % response.url)

        sel = scrapy.Selector(response)
        for h3 in response.xpath('//h3').extract():
            yield MyItem(title=h3)

        for url in response.xpath('//a/@href').extract():
            yield scrapy.Request(url, callback=self.parse)
```

### scrapy.contrib.spiders.CrawlSpider

爬取一般网站常用的spider. 定义了一些规则(rule)来提供跟进link的方便的机制. 提供了一个新的属性:

- rules

    一个包含一个(或多个)`Rule`对象的集合(list). 每个`Rule`对爬取网站的动作定义了特定表现. 如果多个rule匹配了相同的链接, 则根据他们在本属性中被定义的顺序, 第一个会被使用.

也提供了一个可复写(overrideable)的方法:

- parse_start_url(response)

    当start_url的请求返回时, 该方法被调用. 该方法分析最初的返回值并必须返回一个`Item`对象或者一个Request对象或者一个可迭代的包含二者对象.

上面说到的Rule为`scrapy.contrib.spiders.Rule`. 定义方法如下:

```
classscrapy.contrib.spiders.Rule(link_extractor, callback=None, cb_kwargs=None, follow=None, process_links=None, process_request=None)
```

- link_extractor: 是一个`scrapy.contrib.linkextractors.LinkExtractor`对象. 其定义了如何从爬取到的页面提取链接
- callback: 是一个callable或string, 如是string, 该spider中同名的函数将会被调用. 从link_extractor中每获取到链接时将会调用该函数, 该回调函数接受一个`response`作为其第一个参数, 并返回一个包含`Item以及(或)`Request`对象(或者这两者的子类)的列表(list).
    - 避免使用`parse`作为回调函数
- cb_kwargs: 包含传递给回调函数的参数(keyword argument)的字典
- follow: 布尔值, 指定了根据该规则从response提取的链接是否需要跟进. 如果`callback`为None, follow默认设置为True, 否则默认为False
- process_links: 是一个callable或string(该spider中同名的函数将会被调用). 从link_extractor中获取到链接列表时将会调用该函数. 该方法主要用来过滤
- process_request: 是一个callable或string(该spider中同名的函数将会被调用). 该规则提取到每个request时都会调用该函数. 该函数必须返回一个request或者None

样例:

```python
import scrapy
from scrapy.contrib.spiders import CrawlSpider, Rule
from scrapy.contrib.linkextractors import LinkExtractor

class MySpider(CrawlSpider):
    name = 'example.com'
    allowed_domains = ['example.com']
    start_urls = ['http://www.example.com']

    rules = (
        # 提取匹配 'category.php' (但不匹配 'subsection.php') 的链接并跟进链接(没有callback意味着follow默认为True)
        Rule(LinkExtractor(allow=('category\.php', ), deny=('subsection\.php', ))),

        # 提取匹配 'item.php' 的链接并使用spider的parse_item方法进行分析
        Rule(LinkExtractor(allow=('item\.php', )), callback='parse_item'),
    )

    def parse_item(self, response):
        self.log('Hi, this is an item page! %s' % response.url)

        item = scrapy.Item()
        item['id'] = response.xpath('//td[@id="item_id"]/text()').re(r'ID: (\d+)')
        item['name'] = response.xpath('//td[@id="item_name"]/text()').extract()
        item['description'] = response.xpath('//td[@id="item_description"]/text()').extract()
        return item
```

该spider将从example.com的首页开始爬取, 获取category以及item的链接并对后者使用`parse_item`方法. 当item获得返回(response)时, 将使用XPath处理HTML并生成一些数据填入Item中.

#### CrawlSpider工作原理

参考: [Scrapy中的Rules理解](https://blog.csdn.net/wqh_jingsong/article/details/56865433)

CrawlSpider中使用Rule来从当前的网页内容中, 提取下一步需要处理的URL, 并进行下一步处理. 基础的Spider会将得到的response的回调函数指定为`parse`方法. 但CrawlSpider使用Rule抽取下一步需要爬取的URL, 得到爬取结果后, Rule有两套方法进行处理.

```python
def _parse_response(self, response, callback, cb_kwargs, follow=True):
    ##如果传入了callback，使用这个callback解析页面并获取解析得到的reques或item
        if callback:
            cb_res = callback(response, **cb_kwargs) or ()
            cb_res = self.process_results(response, cb_res)
            for requests_or_item in iterate_spider_output(cb_res):
                yield requests_or_item
    ## 其次判断有无follow，用_requests_to_follow解析响应是否有符合要求的link。
        if follow and self._follow_links:
            for request_or_item in self._requests_to_follow(response):
                yield request_or_item
```

上面是CrawlSpider处理每个response的方法
