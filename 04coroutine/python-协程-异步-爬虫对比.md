---
title: 线程，协程对比和 Python 爬虫使用
date: 2019-04-10 10:38:22
tags: Python
---







重点为介绍 `asyncio `。

`asyncio`  可以实现单线程并发 IO 操作。Asynchronous HTTP Client/Server for [asyncio](https://aiohttp.readthedocs.io/en/stable/glossary.html#term-asyncio) and Python。

aiohttp `则是基于` asyncio 实现的 HTTP 框架。

- [AIOHTTP](https://aiohttp.readthedocs.io/en/stable/)

我们先安装 `aiohttp`：

```shell
pip install aiohttp
```

编写一个 Flask 服务器，模拟网络爬虫场景

```python
from flask import Flask
import time

app = Flask(__name__)
@app.route('/')
def index():
    time.sleep(3)
    return 'Hello ,world!'



@app.route('/go')
def go():
    time.sleep(3)
    return 'Hello ,go!'


@app.route('/python')
def python():
    time.sleep(3)
    return 'Hello ,python!'


@app.route('/c')
def c():
    time.sleep(3)
    return 'Hello ,c!'


if __name__ == '__main__':
    app.run(threaded=True, port=5000)

```





多线程协程爬虫，测试是并行

```python
import time
import requests
import asyncio
from multiprocessing.dummy import Pool as ThreadPool

path = ["/go", "/c", "/python", "/"] * 100
host = "http://127.0.0.1:5000"
full_url = list(map(lambda x: f"{host}{x}", path))

thread = 4


async def get_page(url, loop):
    future = loop.run_in_executor(
        None, requests.get, url
    )
    response = await future
    print(response.text)


def divide(i):
    import asyncio
    loop = asyncio.new_event_loop()
    asyncio.set_event_loop(loop)

    tasks = [asyncio.ensure_future(get_page(url, loop))
             for url in full_url]
    loop.run_until_complete(asyncio.wait(tasks))
    loop.close()


if __name__ == '__main__':
    start = time.time()
    pool = ThreadPool(thread)
    # i = [j for j in range(0, thread)]
    pool.map(divide, )
    pool.close()
    pool.join()
    print("爬取{0}个网页 ，总花费时间:{1:.2f}s".format(
        len(path), start - time.time()), end="")


```





#### 异步协程协程 request 代码

```python
import time
import requests
import asyncio

path = ["/go", "/c", "/python", "/"] * 100
host = "http://127.0.0.1:5000"
full_url = list(map(lambda x: f"{host}{x}", path))


async def get_page(url, loop):
    future = loop.run_in_executor(
        None, requests.get, url
    )
    response = await future
    # print(response.text)


if __name__ == '__main__':
    start = time.time()
    loop = asyncio.get_event_loop()
    tasks = [asyncio.ensure_future(get_page(url, loop)) for url in full_url]
    loop.run_until_complete(asyncio.wait(tasks))
    loop.close()
    print("爬取{0}个网页 ，总花费时间:{1:.2f}s".format(
        len(path), start - time.time()), end="")

```

输出结果: 爬取400个网页 ，总花费时间:-21.20s



#### 异步协程爬虫

因 requests 不支持异步。换 `asyncio`

代码代码如下：

```python
import time
import aiohttp
import asyncio


async def get_page(url):
    async with  aiohttp.ClientSession() as session:
        async with await session.get(url=url) as response:
            page_text = await response.text()
            print(page_text)


path = ["/go", "/c", "/python", "/"] * 100
url = "http://127.0.0.1:5000"
full_url = list(map(lambda x: url + x, path))

#
if __name__ == '__main__':
    print(full_url)
    start = time.time()
    loop = asyncio.get_event_loop()
    tasks = [asyncio.ensure_future(get_page(url)) for url in full_url]

    loop.run_until_complete(asyncio.wait(tasks))
    print("爬取{0}个网页 ，总花费时间:{1:.2f}s".format(
        len(path), start - time.time()), end="")



```

输出结果: 爬取400个网页 ，总花费时间:-3.35s



####  如何实现数据解析---任务的绑定回调机制

```python
### 如何实现数据解析---任务的绑定回调机制
import aiohttp
import asyncio
#回调函数：解析响应数据
def callback(task):
    print('this is callback()')
    #获取响应数据
    page_text = task.result()
    print('在回调函数中，实现数据解析')
    
async def get_page(url):
    async with aiohttp.ClientSession() as session:
        async with await session.get(url=url) as response:
            page_text = await response.text() #read()  json()
#             print(page_text)
            return page_text
start = time.time()
path = ["/go", "/c", "/python", "/"] * 100
host = "http://127.0.0.1:5000"
full_url = list(map(lambda x: f"{host}{x}", path))


tasks = []
loop = asyncio.get_event_loop()
for url in urls:
    c = get_page(url)
    task = asyncio.ensure_future(c)
    #给任务对象绑定回调函数用于解析响应数据
    task.add_done_callback(callback)
    tasks.append(task)
loop.run_until_complete(asyncio.wait(tasks))
print('总耗时：',time.time()-start)
```





结合代理池，爬虫速度能做到极致。



#### 参考资料：

- [线程，协程对比和Python爬虫实战说明](https://github.com/zhang0peter/python-coroutine)
- [asyncio：高性能异步模块使用介绍](https://mp.weixin.qq.com/s?__biz=MjM5MzgyODQxMQ==&mid=2650368555&idx=1&sn=a449f107c9c16466c51ce8a6939fcb1b&chksm=be9cd17f89eb5869c00e964e42e79400d4c9b993c4c5764ddbf9ef0e4b85741fc4ab05c77dbc&mpshare=1&scene=23&srcid=07163jZEvRwfwwii9F8dKopl#rd)

- [Python中异步协程的使用方法介绍](https://cuiqingcai.com/6160.html)
- [在 Python 中按需处理数据，第 3 部分 协程和 asyncio](https://www.ibm.com/developerworks/cn/analytics/library/ba-on-demand-data-python-3/index.html)
- [How could I use requests in asyncio?](https://stackoverflow.com/questions/22190403/how-could-i-use-requests-in-asyncio)