---
title: "Python3 学习笔记（异步爬虫）"
date: 2019-04-21T21:06:02+08:00
tags: ["python", "scraping"]
categories: ["Learn", "Python"]
draft: false
---

异步爬虫不同于多进程爬虫，它使用单线程(即仅创建一个事件循环，然后把所有任务添加到事件循环中)就能并发处理多任务。在轮询到某个任务后，当遇到耗时操作(如请求URL)时，挂起该任务并进行下一个任务，当之前被挂起的任务更新了状态(如获得了网页响应)，则被唤醒，程序继续从上次挂起的地方运行下去。极大的减少了中间不必要的等待时间。

有了`Asyncio`异步IO库实现协程后，我们还需要实现异步网页请求。普通的爬虫程序会经常用到requests库用以请求网页并获得服务器响应。而在协程中，由于requests库提供的相关方法不是可等待对象（awaitable），使得无法放在`await`后面，因此无法使用requests库在协程程序中实现请求。

官方专门提供了一个`aiohttp`库，用来实现异步网页请求等功能，简直就是异步版的requests库。在协程中使用`ClientSession()`的`get()`或`request()`方法来请求网页（其中`async with`是异步上下文管理器，其封装了异步实现等功能），完整的`aiohttp`使用方法，请见[官方文档](https://aiohttp.readthedocs.io/en/stable/client_quickstart.html)。

```python
import aiohttp
async with aiohttp.ClientSession() as session:
    async with session.get('http://httpbin.org/get') as resp:
        print(resp.status)
        print(await resp.text())
```

官方API提供的HTTP常见方法：

```python
session.post('http://httpbin.org/post', data=b'data')
session.put('http://httpbin.org/put', data=b'data')
session.delete('http://httpbin.org/delete')
session.head('http://httpbin.org/get')
session.options('http://httpbin.org/get')
session.patch('http://httpbin.org/patch', data=b'data')
```

实例：获取各主要集团招投标平台的标题

```python
from bs4 import BeautifulSoup as bs
import requests
import time
import aiohttp
import asyncio

allBids = {
    '采购与招标网': 'https://www.chinabidding.cn/',
    '招投标平台': 'http://bulletin.cebpubservice.com/',
    '华能招标网': 'http://ec.chng.com.cn/ecmall/',
    '神华招标网': 'http://www.shenhuabidding.com.cn/bidweb/',
    '中广核招标网': 'https://ecp.cgnpc.com.cn/',
    '三峡招标网': 'http://epp.ctg.com.cn/',
    '大唐招标网': 'http://www.cdt-ec.com/home/',
    '守正招标网': 'https://szecp.crc.com.cn/',
    '国电投招标网': 'http://www.cpeinet.com.cn/',
    '国电招标网': 'http://www.cgdcbidding.com/',
    '华电招标网': 'https://www.chdtp.com/',
    '河北招标网': 'http://hebeibidding.com/TPFront/default.aspx',
    '协合招标网': 'http://www.cnegroup.com/'
}
headers = {'User-Agent': 'Mozilla/4.0 (compatible; MSIE 5.5; Windows NT)'}
sem = asyncio.Semaphore(5)  # 限制同时运行协程数量

# 按顺序获取标题
def get_title_normal():
    session = requests.session()
    for name, url in allBids.items():
        html = session.get(url, headers=headers)
        html.encoding = 'utf-8'
        title = bs(html.text, 'lxml').find('title').text
        print(title)

# 协程异步获取标题
async def get_title_asyn(url):
    with (await sem):
        async with aiohttp.ClientSession() as session:  # 获取session
            async with session.get(url, headers=headers) as resp:  # 提出请求
                # 断言，判断网站状态
                assert resp.status == 200
                html = await resp.text()
                soup = bs(html, 'lxml')
                title = soup.find('title').text
                print(''.join(title))


def loops():
    loop = asyncio.get_event_loop()  # 获取事件循环
    # 把所有任务放到一个列表中
    tasks = [get_title_asyn(url) for name, url in allBids.items()]
    loop.run_until_complete(asyncio.wait(tasks))  # 激活协程
    loop.close()  # 关闭事件循环


if __name__ == '__main__':
    print(time.strftime('%Y-%m-%d %H:%M:%S'))
    start = time.time()
    # get_title_normal()
    loops()
    print('总耗时: %.2f秒' % float(time.time()-start))
```
