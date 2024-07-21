---
title: python爬虫基础
date: 2020-06-30 20:07:42
tags: [python]
categories: 后端学习
toc: true
---

爬虫：模拟客户端（浏览器），发送网络请求

入门阶段，还有很多东西需要深究、探索

## 前提

### 要知道

- url = 请求协议 + 网站的域名 + 资源的路径 + 参数
- 浏览器请求的 url 地址，拿到的 response 里面有一些 js、css、html 等内容。也就是所一条 url 请求返回的东西有很多<!-- more -->
- 浏览器中 element 中的内容和爬虫中获得的 url 地址响应不同，所以需要以当前的 url 地址对应的响应为准。在 network 中可以找到，点击里面的 response（页面上点击右键显示网页源码
- http：超文本传输协议。以明文的形式传输，效率高但是不安全
- https：http + ssl（安全套接字层
  - 传输之前数据加密，后来再通过解密获取内容。效率低但是安全

### 前提知识

**http 协议 - 请求**

- 请求行：浏览器中可以找到的那个 request
- 请求头
  - user-agent：用户代理，服务器通过这个知道在请求资源的是什么设备、平台、浏览器
    - 如果是模拟手机来请求服务器，就得使用手机的 user-agent
  - cookie：用来存储用户信息，每次请求都传给服务器
    - 一般在面对要登录的页面的时候会使用到 cookie
    - 这样就在一定程度上防止浏览器知道是在爬虫
- 请求体：_get_ 请求没有，_post_ 请求有
  - 就是携带数据的，get 请求的数据在 url 上，post 的数据在请求体里
  - post 能携带的数据量比较大，t 一般用于登录注册、用于传输大文本的时候

**http 响应**

- 响应头：set-cookie，对方服务器通过这个字段设置 cookie 到本地
  - 等号前的是 cookie 的 _name_ ，后的是 cookie 的 _value_，分号等额每一条 cookie
- 响应体：url 地址对应的响应

## requests

python 爬虫的关键所在，第三方模块。

安装：

```
$ pip install requests
```

使用：直接引入

```python
import requests
```

发送 _get_、_post_ 请求

- response = requests.get( url )

- response = requests.post( url, data={ 请求体的字典 } )

  ```python
  url = "xxxxx"
  query_string = {
      "xxx": "zzz",
      "hhh": "sss"
  }
  headers = {
      "user-agent": "qweasdzxc",
      "cookie": "ahhxixilala"
  }
  response = requests.post(url, data=quert_string, headers=headers)
  print(response.content.decode())
  ```

  - 除了使用 `cookie` 来记录用户，还可以使用 `session` 来记录用户

**response**的方法

- response.text()：这个方法如果出现乱码，就使用 `response.encoding="utf-8"`

  ```python
  # 获取网页的html字符串
  response = requests.get(url)
  # 返回的是随机的编码，一般情况下直接 .text 可以得到，如果不行，就加上这个编码
  response.encoding = "utf-8"
  print(response.text)
  ```

- response.content.decode()：把响应的二进制字节流转化为 _str_ 类型

- response.request.url：发送请求的 ur l 地址

- response.url：response 响应的 url 地址

- response.request.headers：请求头

- response.headers：响应请求

获取源码的方式（这三种方式一定能获得网页正确解码后的字符串）

- response.content.decode()
- response.content.decode("gbk")
- response.text

## requests.session

这里是使用了 session 来代替了 cookie

- 实例化 session

  ```python
  session = requests.session()
  ```

- 例子

  ```python
  # 使用 session 发送 post 请求，获取服务器在本地保存的 cookie，比如登录
  post_url = "xxxssswww"
  headers = {
      "user-agent": "zxcasdqwe"
  }
  post_data = {"username": "jerry", "password": "123456"}
  session.post(post_url, headers=headers, data=post_data)

  # 可再次使用session，请求登录之后的页面
  url = "xxxxxx"
  response = session.get(url, headers=headers)

  # 将内容保存到一个新的文件
  with open("test.html", "w", encoding="utf-8") as f:
      f.write(response.content.decode())
  ```

## 数据提取方法

### json

一种数据交换的格式，一般吧网页切换成 _手机版_ 会返回 json 类型的数据。导入：`import json`，内部模块，不需要使用 pip 下载

- `json.load` 把 json 字符串转化为 python 类型
  - json.load(json 字符串)
- `json.dumps` 把 python 类型转化为字符串
  - json.dumps( { } )
    - json.dumps( val, ensure_ascii=False, indent=2 )
      - ensure_ascii=False：让中文显示成中文，False 首字母要大写
      - indent：让下一行在上一行的基础上空格

### xpath 和 lxml

**xpath**

​ **xpath helper** ：一款浏览器插件，可以在浏览器的 `elements`中定位数据

使用方法：

- ` /html/head/meta`：选中 html 中 head 下的所有 meta 标签
- `//` ：从任意节点开始选择
- `//li` ：当前页面上的所有 li 标签
- `/html/head//link` ：head 下的所有 link 标签
- `@`
  - 选择具体某个元素：`//div[@class='app']/ul/li`（选择了 class 名为 app 下的 ul 下的 li）
  - `@a/@href`：选择 a 标签下的 href 值
- 获取文本：
  - `/a/text()`：获取 a 下面的文本
  - `/a//text()`：获取 a 下面的所有文本
- `.`
  - `./a` ：当前节点下的 a 标签

**lxml**

安装：`$ pip install lxml`

使用：

```python
from lxml import etree
element = etree.HTML("html字符串")
element.xpath("")
```

例如 1

```python
html_str = response.content.decode()
html = etree.HTML(html_str)
print(html)
img_list = html.xpath("//div[@class='indent']/div/table//a[@class='nbg']/img/@src")  # 获取某个页面内的图片
print(img_list)
```

例如 2，获取网页上的多种类型数据(接上)

```python
ret1 = html.xpath("//div[@class='indent']/div/table")  # 设置前缀，后面的使用就不需要重新写这些相同的部分
print(ret1)
for table in ret1:
    item = {}
    # 起初是得到很多数据，后面使用[0].replace.....过滤数据，当然可以使用其他办法，这里使用这个简单的方法
    # [0] 所示投机取巧，不是什么时候都能用
    item["title"] = table.xpath(".//div[@class='pl2']/a//text()")[0].replace("/", "").strip()
    item["href"] = table.xpath(".//div[@class='pl2']/a/@href")[0]
    item["img"] = table.xpath(".//a[@class='nbg']/img/@src")[0]
    item["comment_num"] = table.xpath(".//span[@class='pl']/text()")[0]
    print(item)
```

- strip() 方法用于移除字符串头尾指定的字符（默认为空格或换行符）或字符序列

## 例子

前提安装：requests，lxml

> 网页数据的结构可能已经改变，了解爬虫思路就好了

```python
import requests
from lxml import etree
import json

'''
先定义好思路，然后将思路变成具体的实现方法
'''

class QiuShi:
    def __init__(self):
        self.url_temp = "https://www.qiushibaike.com/text/page/{}/"
        self.headers = {"User-Agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, "
                                      "like Gecko) Chrome/83.0.4103.116 Safari/537.36"}

    def get_url_list(self):
        url_list = [self.url_temp.format(i) for i in range(1, 14)]
        return url_list

    def parse_url(self, url):
        response = requests.get(url, headers=self.headers)
        return response.content.decode()

    def get_content_list(self, html_str):
        html = etree.HTML(html_str)
        # 1. 分组
        div_list = html.xpath("//div[@id='content-left']/div")
        content_list = []
        for div in div_list:
            item = {}
            item["author_name"] = div.xpath("./h2/text()")[0] if len(div.xpath("./h2/text()") > 0) else None
            item["content"] = div.xpath(".//div[@class='content']/span/text()")
            item["stats_vote"] = div.xpath(".//span[@class='stats_vote']/i/text()")
            item["stats_vote"] = item["stats_vote"][0] if len(item["stats_vote"]) > 0 else None
            content_list.append(item)
        return content_list

    def save_content_list(self, content_list):
        with open("qiubai.txt", "a", encoding="utf-8") as f:
            for content in content_list:
                f.write(json.dumps(content, ensure_ascii=False))
                f.write("\n")
            print("保存成功")

    def run(self):
        """实现主要逻辑"""
        # 1. 根据url的规律，构造url list
        url_list = self.get_url_list()
        # 2. 发送请求，获取响应
        for url in url_list:
            html_str = self.parse_url(url)
            # 3. 提取数据
            content_list = self.get_content_list(html_str)
            # 4. 保存
            self.save_content_list(content_list)


if __name__ == '__main__':
    qiushi = QiuShi()
    qiushi.run()

```

## 小结

1. 知道要爬取的 url 和 url 的规律，比如翻页什么的、
2. 使用 requests 发送请求，获取响应
3. 提取数据
   1. 返回 json 字符串，使用 json 模块
   2. 返回的是 html 字符串，使用 lxml 模块 搭配 xpath 提取数据
4. 保存数据

---

加油，脚踏实地。
