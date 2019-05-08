今日概要

- 基于终端指令的持久化存储
- 基于管道的持久化存储

今日详情

1.基于终端指令的持久化存储

- 保证爬虫文件的parse方法中有可迭代类型对象（通常为列表or字典）的返回，该返回值可以通过终端指令的形式写入指定格式的文件中进行持久化操作。

```
执行输出指定格式进行存储：将爬取到的数据写入不同格式的文件中进行存储
    scrapy crawl 爬虫名称 -o xxx.json
    scrapy crawl 爬虫名称 -o xxx.xml
    scrapy crawl 爬虫名称 -o xxx.csv
```



![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

2.基于管道的持久化存储

scrapy框架中已经为我们专门集成好了高效、便捷的持久化操作功能，我们直接使用即可。要想使用scrapy的持久化操作功能，我们首先来认识如下两个文件：

```
    items.py：数据结构模板文件。定义数据属性。
    pipelines.py：管道文件。接收数据（items），进行持久化操作。

持久化流程：
    1.爬虫文件爬取到数据后，需要将数据封装到items对象中。
    2.使用yield关键字将items对象提交给pipelines管道进行持久化操作。
    3.在管道文件中的process_item方法中接收爬虫文件提交过来的item对象，然后编写持久化存储的代码将item对象中存储的数据进行持久化存储
    4.settings.py配置文件中开启管道
```



![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

小试牛刀：将糗事百科首页中的段子和作者数据爬取下来，然后进行持久化存储

\- 爬虫文件：qiubaiDemo.py

```python
# -*- coding: utf-8 -*-
import scrapy
from secondblood.items import SecondbloodItem

class QiubaidemoSpider(scrapy.Spider):
    name = 'qiubaiDemo'
    allowed_domains = ['www.qiushibaike.com']
    start_urls = ['http://www.qiushibaike.com/']

    def parse(self, response):
        odiv = response.xpath('//div[@id="content-left"]/div')
        for div in odiv:
            # xpath函数返回的为列表，列表中存放的数据为Selector类型的数据。我们解析到的内容被封装在了Selector对象中，需要调用extract()函数将解析的内容从Selecor中取出。
            author = div.xpath('.//div[@class="author clearfix"]//h2/text()').extract_first()
            author = author.strip('\n')#过滤空行
            content = div.xpath('.//div[@class="content"]/span/text()').extract_first()
            content = content.strip('\n')#过滤空行

            #将解析到的数据封装至items对象中
            item = SecondbloodItem()
            item['author'] = author
            item['content'] = content

            yield item#提交item到管道文件（pipelines.py）
```



![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

\- items文件：items.py

```python
import scrapy


class SecondbloodItem(scrapy.Item):
    # define the fields for your item here like:
    # name = scrapy.Field()
    author = scrapy.Field() #存储作者
    content = scrapy.Field() #存储段子内容
```



![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

\- 管道文件：pipelines.py

```python
# -*- coding: utf-8 -*-

# Define your item pipelines here
#
# Don't forget to add your pipeline to the ITEM_PIPELINES setting
# See: https://doc.scrapy.org/en/latest/topics/item-pipeline.html


class SecondbloodPipeline(object):
    #构造方法
    def __init__(self):
        self.fp = None  #定义一个文件描述符属性
　　#下列都是在重写父类的方法：
    #开始爬虫时，执行一次
    def open_spider(self,spider):
        print('爬虫开始')
        self.fp = open('./data.txt', 'w')

　　 #因为该方法会被执行调用多次，所以文件的开启和关闭操作写在了另外两个只会各自执行一次的方法中。
    def process_item(self, item, spider):
        #将爬虫程序提交的item进行持久化存储
        self.fp.write(item['author'] + ':' + item['content'] + '\n')
        return item

    #结束爬虫时，执行一次
    def close_spider(self,spider):
        self.fp.close()
        print('爬虫结束')
```



![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

\- 配置文件：settings.py

```python
#开启管道
ITEM_PIPELINES = {
    'secondblood.pipelines.SecondbloodPipeline': 300, #300表示为优先级，值越小优先级越高
}
```



![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

2.1 基于mysql的管道存储

小试牛刀案例中，在管道文件里将item对象中的数据值存储到了磁盘中，如果将item数据写入mysql数据库的话，只需要将上述案例中的管道文件修改成如下形式：

\- pipelines.py文件

```python
# -*- coding: utf-8 -*-

# Define your item pipelines here
#
# Don't forget to add your pipeline to the ITEM_PIPELINES setting
# See: https://doc.scrapy.org/en/latest/topics/item-pipeline.html

#导入数据库的类
import pymysql
class QiubaiproPipelineByMysql(object):

    conn = None  #mysql的连接对象声明
    cursor = None#mysql游标对象声明
    def open_spider(self,spider):
        print('开始爬虫')
        #链接数据库
        self.conn = pymysql.Connect(host='127.0.0.1',port=3306,user='root',password='123456',db='qiubai')
    #编写向数据库中存储数据的相关代码
    def process_item(self, item, spider):
        #1.链接数据库
        #2.执行sql语句
        sql = 'insert into qiubai values("%s","%s")'%(item['author'],item['content'])
        self.cursor = self.conn.cursor()
        #执行事务
        try:
            self.cursor.execute(sql)
            self.conn.commit()
        except Exception as e:
            print(e)
            self.conn.rollback()

        return item
    def close_spider(self,spider):
        print('爬虫结束')
        self.cursor.close()
        self.conn.close()
```



![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

\- settings.py

```python
ITEM_PIPELINES = {
    'qiubaiPro.pipelines.QiubaiproPipelineByMysql': 300,
}
```



![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

2.2 基于redis的管道存储

小试牛刀案例中，在管道文件里将item对象中的数据值存储到了磁盘中，如果将item数据写入redis数据库的话，只需要将上述案例中的管道文件修改成如下形式：

```python
# -*- coding: utf-8 -*-

# Define your item pipelines here
#
# Don't forget to add your pipeline to the ITEM_PIPELINES setting
# See: https://doc.scrapy.org/en/latest/topics/item-pipeline.html

import redis

class QiubaiproPipelineByRedis(object):
    conn = None
    def open_spider(self,spider):
        print('开始爬虫')
        #创建链接对象
        self.conn = redis.Redis(host='127.0.0.1',port=6379)
    def process_item(self, item, spider):
        dict = {
            'author':item['author'],
            'content':item['content']
        }
        #写入redis中
        self.conn.lpush('data', dict)
        return item
```



![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

\- pipelines.py文件 

```python
ITEM_PIPELINES = {
    'qiubaiPro.pipelines.QiubaiproPipelineByRedis': 300,
}
```



![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

 

\- 面试题：如果最终需要将爬取到的数据值一份存储到磁盘文件，一份存储到数据库中，则应该如何操作scrapy？　　

\- 答：管道文件中的代码为

```python
#该类为管道类，该类中的process_item方法是用来实现持久化存储操作的。
class DoublekillPipeline(object):

    def process_item(self, item, spider):
        #持久化操作代码 （方式1：写入磁盘文件）
        return item

#如果想实现另一种形式的持久化操作，则可以再定制一个管道类：
class DoublekillPipeline_db(object):

    def process_item(self, item, spider):
        #持久化操作代码 （方式1：写入数据库）
        return item
```



![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

 在settings.py开启管道操作代码为：

```python
#下列结构为字典，字典中的键值表示的是即将被启用执行的管道文件和其执行的优先级。
ITEM_PIPELINES = {
   'doublekill.pipelines.DoublekillPipeline': 300,
    'doublekill.pipelines.DoublekillPipeline_db': 200,
}

#上述代码中，字典中的两组键值分别表示会执行管道文件中对应的两个管道类中的process_item方法，实现两种不同形式的持久化操作。
```



![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)