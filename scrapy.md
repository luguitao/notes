##### 创建爬虫项目
安装完scrapy后, 使用如下指令
```shell
scrapy startproject <projectname>
```
就可以创建一个对应名字的爬虫项目
```shell
tutorial/
    scrapy.cfg            # deploy configuration file
    tutorial/             # project's Python module, you'll import your code from here
        __init__.py
        items.py          # project items definition file
        pipelines.py      # project pipelines file
        settings.py       # project settings file
        spiders/          # a directory where you'll later put your spiders
            __init__.py
```

##### 第一个爬虫
爬虫就是scrapy用来从网页抓取信息的类, 必须是scrapy.Spider的子类, 定义怎么初始化请求, 怎么去跟踪网页的连接, 怎么去处理响应回来的网页信息.
只要在我们在上一步新建的爬虫项目目录下, 使用如下指令
```
scrapy genspider <spidername> <targetDomainName> [--template=<templateName>]
```
就可以在项目下的spiders路径下, 新建一个爬虫类, 比如
```
scrapy genspider patant baidu.com --template=crawl
```
就可以使用内置的crawl模板, 在项目名/spiders下, 生成一个patant.py的爬虫类
```
import scrapy


class QuotesSpider(scrapy.Spider):
    name = "quotes"

    def start_requests(self):
        urls = [
            'http://quotes.toscrape.com/page/1/',
            'http://quotes.toscrape.com/page/2/',
        ]
        for url in urls:
            yield scrapy.Request(url=url, callback=self.parse)

    def parse(self, response):
        page = response.url.split("/")[-2]
        filename = 'quotes-%s.html' % page
        with open(filename, 'wb') as f:
            f.write(response.body)
        self.log('Saved file %s' % filename)
```
* name: spider的名字, 必须唯一
* start_requests(): 必须返回一组可迭代的请求(返回一个请求列表, 或者写一个生成请求的方法), 爬虫会从这组请求开始爬取, 后续的请求会通过这些初始请求生成. 也可以不实现此方法, 而改为定义一个start_urls列表, start_urls=['url1', 'url2', 'url3',], 这个列表会作为默认实现的start_requests()的返回值
* parse(): 对发送请求后, 响应回来的页面信息进行处理, 返回一个信息的字典, 创建信息的请求供进一步爬取. 它的参数是TextResponse类的一个实例, 包含信息本身和一些有用的方法. 即便不在start_requests()方法中显式调用parse()方法, parse()本身也会作为默认回调方法被调用

##### 运行爬虫
```
scrapy crawl <spiderName>
```
爬虫依次爬取start_request()方法返回的每一个请求, 每收到一个响应就实例化一个Response对象, 传给parse()方法

##### 提取数据
可以用scrapy自带的交互命令行调试信息抓取, 可以使用如下命令
```
scrapy shell 'http://quotes.toscrape.com/page/1/' #官方文档格式
scrapy shell quotes.toscrape.com/page/1/ #实践中发现会自动补全http://, 所以不用带协议头也不用带引号, 但是这样就不能传入其他参数了
scrapy shell "http://quotes.toscrape.com/page/1/" #官方文档格式, 推荐windows用户可以用双引号
```
可以尝试CSS选择器, 返回一个列表
```
>>> response.css('title')
[<Selector xpath='descendant-or-self::title' data='<title>Quotes to Scrape</title>'>]
```
可以进一步操作, 如果不加::text, 则会连带标签一起返回. 如果extract的对象是一个列表, 也会返回一个列表
```
>>> response.css('title::text').extract()
['Quotes to Scrape']
```
如果只想返回第一个值
```
response.css('title::text').extract_first() #推荐, 如果找不到会返回None
response.css('title::text')[0].extract() #不推荐, 如果找不到会报数组下标越界
```
还可以通过正则来提取, 用re()方法
```
>>> response.css('title::text').re(r'Quotes.*')
['Quotes to Scrape']
>>> response.css('title::text').re(r'Q\w+')
['Quotes']
>>> response.css('title::text').re(r'(\w+) to (\w+)')
['Quotes', 'Scrape']
```
比如我们的目标网站, 可以通过css和xpath结合, 来获取下一步访问的请求地址
```
scrapy shell "http://appft.uspto.gov/netacgi/nph-Parser?Sect1=PTO2&Sect2=HITOFF&p=1&u=%2Fnetahtml%2FPTO%2Fsearch-bool.html&r=0&f=S&l=50&co1=AND&d=PG01&s1=nitrogen&s2=wastewater&Page=Prev&OS=nitrogen%2520AND%2520wastewater&RS=nitrogen%2520AND%2520wastewater"
>>> response.css('td a').xpath('@href')[12].extract()
u'/netacgi/nph-Parser?Sect1=PTO2&Sect2=HITOFF&p=1&u=%2Fnetahtml%2FPTO%2Fsearch-bool.html&r=7&f=G&l=50&co1=AND&d=PG01&s1=nitrogen&s2=wastewater&OS=nitrogen+AND+wastewater&RS=nitrogen+AND+wastewater'
```

爬虫可以生成多个字典, 通过yield关键字返回.
yield关键词代表该方法是一个生成器, 不调用next()方法时会挂起, 调用则返回方法体中执行完第1个yield语句, 再挂起; 再调用next()则会执行到第2个yield语句为止, 以此类推.
```
import scrapy


class QuotesSpider(scrapy.Spider):
    name = "quotes"
    start_urls = [
        'http://quotes.toscrape.com/page/1/',
        'http://quotes.toscrape.com/page/2/',
    ]

    def parse(self, response):
        for quote in response.css('div.quote'):
            yield {
                'text': quote.css('span.text::text').extract_first(),
                'author': quote.css('small.author::text').extract_first(),
                'tags': quote.css('div.tags a.tag::text').extract(),
            }
```
返回值为
```
2016-09-19 18:57:19 [scrapy.core.scraper] DEBUG: Scraped from <200 http://quotes.toscrape.com/page/1/>
{'tags': ['life', 'love'], 'author': 'André Gide', 'text': '“It is better to be hated for what you are than to be loved for what you are not.”'}
2016-09-19 18:57:19 [scrapy.core.scraper] DEBUG: Scraped from <200 http://quotes.toscrape.com/page/1/>
{'tags': ['edison', 'failure', 'inspirational', 'paraphrased'], 'author': 'Thomas A. Edison', 'text': "“I have not failed. I've just found 10,000 ways that won't work.”"}
```

##### 数据存储
最简单的数据存储是用Feed exports, 常用的形式用四种, json, json lines, xml, csv
```
scrapy crawl quotes -o quotes.json
```
可以打印出抓取到的所有item的json文件
因为历史原因, 输出会追加在文件中而不是复写, 所以执行两遍的话, json格式会被破坏, 用json lines就会安全很多, 而且它是流式的
```
scrapy crawl quotes -o quotes.jl
```
如果要对抓取的item做复杂处理, 可以在pupelines.py做相应处理

##### 二级链接
如果想在爬取start_urls后获得数据中获取二级链接进行进一步爬取, 则可以通过命令
```
>>> response.css('li.next a::attr(href)').extract_first()
```
提取相应中的url地址, 作为二级链接进行爬取
```
import scrapy


class QuotesSpider(scrapy.Spider):
    name = "quotes"
    start_urls = [
        'http://quotes.toscrape.com/page/1/',
    ]

    def parse(self, response):
        for quote in response.css('div.quote'):
            yield {
                'text': quote.css('span.text::text').extract_first(),
                'author': quote.css('small.author::text').extract_first(),
                'tags': quote.css('div.tags a.tag::text').extract(),
            }

        next_page = response.css('li.next a::attr(href)').extract_first()
        if next_page is not None:
            next_page = response.urljoin(next_page)
            yield scrapy.Request(next_page, callback=self.parse)
```
链接可能是相对路径, 所以用urljoin()方法补全路径.
递归调用parse()方法自身.

##### 建立请求的捷径
