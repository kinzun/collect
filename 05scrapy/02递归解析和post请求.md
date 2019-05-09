



代码展示：

```python
# -*- coding: utf-8 -*-
import scrapy
from qiushibaike.items import QiushibaikeItem
# scrapy.http import Request
class QiushiSpider(scrapy.Spider):
    name = 'qiushi'
    allowed_domains = ['www.qiushibaike.com']
    start_urls = ['https://www.qiushibaike.com/text/']

    #爬取多页
    pageNum = 1 #起始页码
    url = 'https://www.qiushibaike.com/text/page/%s/' #每页的url

    def parse(self, response):
        div_list=response.xpath('//*[@id="content-left"]/div')
        for div in div_list:
            #//*[@id="qiushi_tag_120996995"]/div[1]/a[2]/h2
            author=div.xpath('.//div[@class="author clearfix"]//h2/text()').extract_first()
            author=author.strip('\n')
            content=div.xpath('.//div[@class="content"]/span/text()').extract_first()
            content=content.strip('\n')
            item=QiushibaikeItem()
            item['author']=author
            item['content']=content

            yield item #提交item到管道进行持久化

         #爬取所有页码数据
        if self.pageNum <= 13: #一共爬取13页（共13页）
            self.pageNum += 1
            url = format(self.url % self.pageNum)

            #递归爬取数据：callback参数的值为回调函数（将url请求后，得到的相应数据继续进行parse解析），递归调用parse函数
            yield scrapy.Request(url=url,callback=self.parse)
```



![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

2.五大核心组件工作流程：



![](../../the_picture/scrapy.jpg)



- ### Scrapy Engine
  
  引擎负责控制数据流在系统中所有组件中流动，并在相应动作发生时触发事件。 详细内容查看下面的数据流( Data Flow )部分。
  
  ### 调度器(Scheduler)
  
  调度器从引擎接受 request 并将他们入队，以便之后引擎请求他们时提供给引擎。
  
  ### 下载器(Downloader)
  
  下载器负责获取页面数据并提供给引擎，而后提供给 spider。
  
  ### Spiders
  
  Spider 是 Scrapy 用户编写用于分析 response 并提取item(即获取到的item)或额外跟进的URL的类。 每个 spider 负责处理一个特定(或一些)网站。 更多内容请看 [Spiders](https://scrapy-chs.readthedocs.io/zh_CN/0.24/topics/spiders.html#topics-spiders) 。
  
  ### Item Pipeline
  
  Item Pipeline 负责处理被spider提取出来的 item。典型的处理有清理、 验证及持久化(例如存取到数据库中)。 更多内容查看 [Item Pipeline](https://scrapy-chs.readthedocs.io/zh_CN/0.24/topics/item-pipeline.html#topics-item-pipeline) 。



## 数据流(Data flow)

Scrapy中的数据流由执行引擎控制，其过程如下:

1. 引擎打开一个网站( open a domain )，找到处理该网站的 Spider 并向该spider 请求第一个要爬取的URL(s)。
2. 引擎从Spider中获取到第一个要爬取的URL并在调度器( Scheduler )以Request 调度。
3. 引擎向调度器请求下一个要爬取的 URL。
4. 调度器返回下一个要爬取的URL给引擎，引擎将URL通过下载中间件(请求( request )方向)转发给下载器( Downloader )。
5. 一旦页面下载完毕，下载器生成一个该页面的 Response，并将其通过下载中间件(返回( response )方向)发送给引擎。
6. 引擎从下载器中接收到 Response 并通过Spider中间件(输入方向)发送给Spider 处理。
7. Spider 处理 Response 并返回爬取到的 Item 及(跟进的)新的 Request 给引擎。
8. 引擎将( Spider 返回的)爬取到的 Item 给 Item Pipeline，将( Spider 返回的) Request 给调度器。
9. (从第二步)重复直到调度器中没有更多地 request，引擎关闭该网站。



递归解析请求发送



```python
  def start_requests(self):
        for u in self.start_urls:
           yield scrapy.Request(url=u,callback=self.parse)
```



## Post 

方法： 重写start_requests方法，让其发起post请求：

```python
def start_requests(self):
        #请求的url
        post_url = 'http://fanyi.baidu.com/sug'
        # post请求参数
        formdata = {
            'kw': 'wolf',
        }
        # 发送post请求
        yield scrapy.FormRequest(url=post_url, formdata=formdata, callback=self.parse)
```