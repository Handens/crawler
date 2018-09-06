# day10

### 1,CrawlSpider

他是一个类，scrapy里面有好多的爬虫类推，基类就是spider，CrawlSpider也是一个爬虫类，是spider的子类。Crawlspider比spider强大强大在可以提权链接，通过一个对象的方法来提取链接，写一个规则提取符合规则的链接。

链接提取器	LinkExtractor()

创建对象过程

​	le = LinkExtractor(allow=xxx,restrict_xpaths=xxx,restrict_css=xxx)

allow : 正则表达式

restrict_xpaths : 写一个xpath路径

restrict_css : 写一个选择器



