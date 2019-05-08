​	







## 原始响应内容

在罕见的情况下，你可能想获取来自服务器的原始套接字响应，那么你可以访问 `r.raw`。 如果你确实想这么干，那请你确保在初始请求中设置了 `stream=True`。具体你可以这么做：

```Python
>>> r = requests.get('https://api.github.com/events', stream=True)
>>> r.raw
<requests.packages.urllib3.response.HTTPResponse object at 0x101194810>
>>> r.raw.read(10)
'\x1f\x8b\x08\x00\x00\x00\x00\x00\x00\x03'
```

但一般情况下，你应该以下面的模式将文本流保存到文件：

```Python
with open(filename, 'wb') as fd:
    for chunk in r.iter_content(chunk_size):
        fd.write(chunk)
```

使用 `Response.iter_content` 将会处理大量你直接使用 `Response.raw` 不得不处理的。 当流下载时，上面是优先推荐的获取内容方式。 Note that `chunk_size` can be freely adjusted to a number that may better fit your use cases.





##### Python Response.iter_content 代码示例

```python
import requests
import random

from bs4 import BeautifulSoup
from lxml import etree
from tqdm import tqdm


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


def download(url,file_name):

    re = rq_url(url)
    html = etree.HTML(re.text)
    result = html.xpath('//*[@id="down"]/div[2]/ul/li/a')
    download_link = random.choice(result).get('href')

    resp = requests.get(download_link, stream=True, headers=HEADERS)
    #stream=True的作用是仅让响应头被下载，连接保持打开状态，
    print(resp.headers)
    content_size = int(resp.headers['Content-Length'])/1024   #确定整个安装包的大小

    with open("test.rar", 'wb') as fd:
#         for chunk in r.iter_content(chunk_size=1024):
#             fd.write(chunk)
        for data in tqdm(iterable=resp.iter_content(1024),total=content_size,unit='k',desc=file_name):
            fd.write(data)
        print(file_name+ "已经下载完毕！")
        
url = "http://sc.chinaz.com/jianli/190501265061.htm"
name = url.split('/')[-1]    #截取整个url最后一段即文件名
download(url,name))
```



[tqdm](https://rorschachchan.github.io/2018/07/24/%E4%BD%BF%E7%94%A8tqdm%E6%B7%BB%E5%8A%A0%E4%B8%8B%E8%BD%BD%E7%9A%84%E8%BF%9B%E5%BA%A6%E6%9D%A1/)





## [Cookie](http://cn.python-requests.org/zh_CN/latest/user/quickstart.html#cookie)

```
#!/usr/bin/env python
# -*- coding:utf-8 -*-
import requests

post_url = 'http://www.renren.com/ajaxLogin/login?1=1&uniqueTimestamp=201873958471'
session = requests.session()
HEADERS = {"User-Agent": "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_14_2) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/73.0.3683.103 Safari/537.36"}

formdata = {'email': '17701256561','icode': '',}

#使用session发送请求，目的是为了将session保存该次请求中的cookie
session.post(url=post_url,data=formdata,headers=HEADERS)

get_url = 'http://www.renren.com/960481378/profile'
#再次使用session进行请求的发送，该次请求中已经携带了cookie
response = session.get(url=get_url,headers=HEADERS)
#设置响应内容的编码格式
response.encoding = 'utf-8'
#将响应内容写入文件
with open('./renren.html','w') as fp:
	fp.write(response.text)
```



[Requests 中文文档](http://cn.python-requests.org/zh_CN/latest/user/quickstart.html)

[高级用法](https://2.python-requests.org//zh_CN/latest/user/advanced.html)