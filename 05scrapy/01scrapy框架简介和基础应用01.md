- scrapy框架介绍
- 环境安装
- 基础使用



一.什么是Scrapy？

　　Scrapy是一个为了爬取网站数据，提取结构性数据而编写的应用框架，非常出名，非常强悍。所谓的框架就是一个已经被集成了各种功能（高性能异步下载，队列，分布式，解析，持久化等）的具有很强通用性的项目模板。对于框架的学习，重点是要学习其框架的特性、各个功能的用法即可。

二.安装

　　`Linux：`

```
      ``pip3 install scrapy
```

 

```
　　Windows：
      ``a. pip3 install wheel
      ``b. 下载twisted http:``/``/``www.lfd.uci.edu``/``~gohlke``/``pythonlibs``/``#twisted
      ``c. 进入下载目录，执行 pip3 install Twisted‑``17.1``.``0``‑cp35‑cp35m‑win_amd64.whl
      ``d. pip3 install pywin32
      ``e. ``pip3 install scrapy
```

三.基础使用

　　1.创建项目：scrapy startproject 项目名称

　　　　项目结构：

```
project_name/
   scrapy.cfg：
   project_name/
       __init__.py
       items.py
       pipelines.py
       settings.py
       spiders/
           __init__.py

scrapy.cfg   项目的主配置信息。（真正爬虫相关的配置信息在settings.py文件中）
items.py     设置数据存储模板，用于结构化数据，如：Django的Model
pipelines    数据持久化处理
settings.py  配置文件，如：递归的层数、并发数，延迟下载等
spiders      爬虫目录，如：创建文件，编写爬虫解析规则
```



![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

　2.创建爬虫应用程序：

　　　　　　cd project_name（进入项目目录）

　　　　　　scrapy genspider 应用名称 爬取网页的起始url （例如：scrapy genspider qiubai www.qiushibaike.com）

　　3.编写爬虫文件:在步骤2执行完毕后，会在项目的spiders中生成一个应用名的py爬虫文件，文件源码如下：

```
# -*- coding: utf-8 -*-
import scrapy

class QiubaiSpider(scrapy.Spider):
    name = 'qiubai' #应用名称
    #允许爬取的域名（如果遇到非该域名的url则爬取不到数据）
    allowed_domains = ['https://www.qiushibaike.com/']
    #起始爬取的url
    start_urls = ['https://www.qiushibaike.com/']

     #访问起始URL并获取结果后的回调函数，该函数的response参数就是向起始的url发送请求后，获取的响应对象.该函数返回值必须为可迭代对象或者NUll 
     def parse(self, response):
        print(response.text) #获取字符串类型的响应内容
        print(response.body)#获取字节类型的相应内容
```



![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

　　4.设置修改settings.py配置文件相关配置:

```
修改内容及其结果如下：
19行：USER_AGENT = 'Mozilla/5.0 (Macintosh; Intel Mac OS X 10_12_0) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/68.0.3440.106 Safari/537.36' #伪装请求载体身份

22行：ROBOTSTXT_OBEY = False  #可以忽略或者不遵守robots协议
```



![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

​      5.执行爬虫程序：scrapy crawl  应用名称

四.小试牛刀：将糗百首页中段子的内容和标题进行爬取

```
# -*- coding: utf-8 -*-
import scrapy

class QiubaiSpider(scrapy.Spider):
    name = 'qiubai'
    allowed_domains = ['https://www.qiushibaike.com/']
    start_urls = ['https://www.qiushibaike.com/']

    def parse(self, response):
        #xpath为response中的方法，可以将xpath表达式直接作用于该函数中
        odiv = response.xpath('//div[@id="content-left"]/div')
        content_list = [] #用于存储解析到的数据
        for div in odiv:
            #xpath函数返回的为列表，列表中存放的数据为Selector类型的数据。我们解析到的内容被封装在了Selector对象中，需要调用extract()函数将解析的内容从Selecor中取出。
            author = div.xpath('.//div[@class="author clearfix"]/a/h2/text()')[0].extract()
            content=div.xpath('.//div[@class="content"]/span/text()')[0].extract()

            #将解析到的内容封装到字典中
            dic={
                '作者':author,
                '内容':content
            }
            #将数据存储到content_list这个列表中
            content_list.append(dic)

        return content_list
```



![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

执行爬虫程序：

```
    scrapy crawl 爬虫名称 ：该种执行形式会显示执行的日志信息
    scrapy crawl 爬虫名称 --nolog：该种执行形式不会显示执行的日志信息
```