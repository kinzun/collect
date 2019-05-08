# Python 爬虫 三种数据解析方式



- 正则解析
- xpath 解析
- bs4 解析

正则解析

## Xpath解析

- 测试页面数据

```html
<html lang="en">
<head>
	<meta charset="UTF-8" />
	<title>测试bs4</title>
</head>
<body>
	<div>
		<p>百里守约</p>
	</div>
	<div class="song">
		<p>李清照</p>
		<p>王安石</p>
		<p>苏轼</p>
		<p>柳宗元</p>
		<a href="http://www.song.com/" title="赵匡胤" target="_self">
			<span>this is span</span>
		宋朝是最强大的王朝，不是军队的强大，而是经济很强大，国民都很有钱</a>
		<a href="" class="du">总为浮云能蔽日,长安不见使人愁</a>
		<img src="http://www.baidu.com/meinv.jpg" alt="" />
	</div>
	<div class="tang">
		<ul>
			<li><a href="http://www.baidu.com" title="qing">清明时节雨纷纷,路上行人欲断魂,借问酒家何处有,牧童遥指杏花村</a></li>
			<li><a href="http://www.163.com" title="qin">秦时明月汉时关,万里长征人未还,但使龙城飞将在,不教胡马度阴山</a></li>
			<li><a href="http://www.126.com" alt="qi">岐王宅里寻常见,崔九堂前几度闻,正是江南好风景,落花时节又逢君</a></li>
			<li><a href="http://www.sina.com" class="du">杜甫</a></li>
			<li><a href="http://www.dudu.com" class="du">杜牧</a></li>
			<li><b>杜小月</b></li>
			<li><i>度蜜月</i></li>
			<li><a href="http://www.haha.com" id="feng">凤凰台上凤凰游,凤去台空江自流,吴宫花草埋幽径,晋代衣冠成古丘</a></li>
		</ul>
	</div>
</body>
</html>
```

- 常用xpath表达式回顾

```python
属性定位：
    #找到class属性值为song的div标签
    //div[@class="song"] 
层级&索引定位：
    #找到class属性值为tang的div的直系子标签ul下的第二个子标签li下的直系子标签a
    //div[@class="tang"]/ul/li[2]/a
逻辑运算：
    #找到href属性值为空且class属性值为du的a标签
    //a[@href="" and @class="du"]
模糊匹配：
    //div[contains(@class, "ng")]
    //div[starts-with(@class, "ta")]
取文本：
    # /表示获取某个标签下的文本内容
    # //表示获取某个标签下的文本内容和所有子标签下的文本内容
    //div[@class="song"]/p[1]/text()
    //div[@class="tang"]//text()
取属性：
    //div[@class="tang"]//li[2]/a/@href
```

- 代码中使用xpath表达式进行数据解析：

```python
1.下载：pip install lxml
2.导包：from lxml import etree

3.将html文档或者xml文档转换成一个etree对象，然后调用对象中的方法查找指定的节点

　　2.1 本地文件：tree = etree.parse(文件名)
                tree.xpath("xpath表达式")

　　2.2 网络数据：tree = etree.HTML(网页内容字符串)
                tree.xpath("xpath表达式")
```





站长素材代码示例>

```python
import requests
from bs4 import BeautifulSoup
from lxml import etree
import random

HEADERS = {
    "User-Agent": "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_14_2) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/73.0.3683.103 Safari/537.36"}


def rq_url(URL):

    page_html = requests.get(url=URL, headers=HEADERS,)
    page_html.encoding = 'utf-8'

    return page_html


def full_info_func():
    re = rq_url(URL="http://sc.chinaz.com/jianli/free.html")
    html = etree.HTML(re.text)
    result = html.xpath('//*[@id="container"]')[0]
    full_url = []
    for i in result:
        resume_url = i.xpath('./p/a')[0].get("href")
        resume_info = i.xpath('./p/a')[0].text
        full = {"url": resume_url, "title": resume_info}
        full_url.append(full)


def download(url):

    re = rq_url(url)
    html = etree.HTML(re.text)
    result = html.xpath('//*[@id="down"]/div[2]/ul/li/a')
    download_link = random.choice(result).get('href')

    r = requests.get(download_link, stream=True, headers=HEADERS)

    with open("test.rar", 'wb') as fd:
        for chunk in r.iter_content(chunk_size=1024):
            fd.write(chunk)


download("http://sc.chinaz.com/jianli/190501265061.htm")
```



参考链接

- [lxml 学习笔记](https://www.jianshu.com/p/e084c2b2b66d)
- [lxml.etree 官方文档](https://lxml.de/tutorial.html)
- [lxml 译文](https://www.cnblogs.com/cnhkzyy/p/7490292.html)





### BeautifulSoup 解析



- 基础使用

```python
使用流程：       
    - 导包：from bs4 import BeautifulSoup
    - 使用方式：可以将一个html文档，转化为BeautifulSoup对象，然后通过对象的方法或者属性去查找指定的节点内容
        （1）转化本地文件：
             - soup = BeautifulSoup(open('本地文件'), 'lxml')
        （2）转化网络文件：
             - soup = BeautifulSoup('字符串类型或者字节类型', 'lxml')
        （3）打印soup对象显示内容为html文件中的内容

基础巩固：
    （1）根据标签名查找
        - soup.a   只能找到第一个符合要求的标签
    （2）获取属性
        - soup.a.attrs  获取a所有的属性和属性值，返回一个字典
        - soup.a.attrs['href']   获取href属性
        - soup.a['href']   也可简写为这种形式
    （3）获取内容
        - soup.a.string
        - soup.a.text
        - soup.a.get_text()
       【注意】如果标签还有标签，那么string获取到的结果为None，而其它两个，可以获取文本内容
    （4）find：找到第一个符合要求的标签
        - soup.find('a')  找到第一个符合要求的
        - soup.find('a', title="xxx")
        - soup.find('a', alt="xxx")
        - soup.find('a', class_="xxx")
        - soup.find('a', id="xxx")
    （5）find_all：找到所有符合要求的标签
        - soup.find_all('a')
        - soup.find_all(['a','b']) 找到所有的a和b标签
        - soup.find_all('a', limit=2)  限制前两个
    （6）根据选择器选择指定的内容
               select:soup.select('#feng')
        - 常见的选择器：标签选择器(a)、类选择器(.)、id选择器(#)、层级选择器
            - 层级选择器：
                div .dudu #lala .meme .xixi  下面好多级
                div > p > a > .lala          只能是下面一级
        【注意】select选择器返回永远是列表，需要通过下标提取指定的对象
```

- 需求：使用bs4实现将诗词名句网站中三国演义小说的每一章的内容爬去到本地磁盘进行存储   http://www.shicimingju.com/book/sanguoyanyi.html

  ```python
  #!/usr/bin/env python
  # -*- coding:utf-8 -*-
  import requests
  from bs4 import BeautifulSoup
  
  headers={
           'User-Agent': 'Mozilla/5.0 (Macintosh; Intel Mac OS X 10_12_0) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/69.0.3497.100 Safari/537.36',
       }
  def parse_content(url):
      #获取标题正文页数据
      page_text = requests.get(url,headers=headers).text
      soup = BeautifulSoup(page_text,'lxml')
      #解析获得标签
      ele = soup.find('div',class_='chapter_content')
      content = ele.text #获取标签中的数据值
      return content
  
  if __name__ == "__main__":
       url = 'http://www.shicimingju.com/book/sanguoyanyi.html'
       reponse = requests.get(url=url,headers=headers)
       page_text = reponse.text
  
       #创建soup对象
       soup = BeautifulSoup(page_text,'lxml')
       #解析数据
       a_eles = soup.select('.book-mulu > ul > li > a')
       print(a_eles)
       cap = 1
       for ele in a_eles:
           print('开始下载第%d章节'%cap)
           cap+=1
           title = ele.string
           content_url = 'http://www.shicimingju.com'+ele['href']
           content = parse_content(content_url)
  
           with open('./sanguo.txt','w') as fp:
               fp.write(title+":"+content+'\n\n\n\n\n')
               print('结束下载第%d章节'%cap)
  ```

   

 

[](https://www.cnblogs.com/bobo-zhang/p/9680673.html)