---
layout: post
title: "Scrapy_Quick_Start"
date: 2015-07-22 10:39:47 +0800
comments: true
categories: 
---
Scrapy的官网教程十分详细，并且也有中文版，推荐阅读。

[英文版](http://doc.scrapy.org/en/1.0/intro/overview.html)

[中文版](http://scrapy-chs.readthedocs.org/zh_CN/latest/intro/overview.html)
<!-- more -->
## Scrapy环境搭建及简单实例

Scrapy([官网](http://scrapy.org/))是Python环境下的一个Web抓取框架，可以方便的抓取Web页面并且提取结构化数据。

## Win7下搭建Python爬虫库Scrapy步骤

Scrapy官网安装教程([地址](http://doc.scrapy.org/en/1.0/intro/install.html))

步骤

- 安装 Python2.7([地址](https://www.python.org/downloads/))
- 安装 pywin32([地址](http://sourceforge.net/projects/pywin32/files/))
- 安装Python包管理工具 PIP
    1. 下载 pip文件pip-7.1.0.tar.gz([地址](https://pypi.python.org/pypi/pip#downloads))
    2. 下载 setuptools文件setuptools-18.0.1.tar.gz([地址](https://pypi.python.org/pypi/setuptools))
    3. 解压文件到Python安装目录下的 `Lib/site-packages/`下
    4. 进入PIP目录运行`setup.py`安装(`python setup.py install`)
    5. 将Python安装目录下的`\Scripts`加入环境变量Path中
- 安装Scrapy
    1. 命令行运行`pip install scrapy`进行下载安装
    2. 命令行输入`scrapy`，查看安装是否成功

## 使用Scrapy爬虫的一个简单实例

Scrapy官网实例教程([地址](http://doc.scrapy.org/en/1.0/intro/tutorial.html))

使用Scrapy抓取本博客的博文列表

- 创建一个新的Scrapy项目

    - 在一个空目录下打开命令行执行：`scrapy startproject tutorial`，会新生成一个名为tutorial的文件夹
    - 目录结构如下：
    {% codeblock %}
  tutorial/
  scrapy.cfg            # deploy configuration file
  tutorial/             # project's Python module, you'll import your code from here
      __init__.py
      items.py          # project items file
      pipelines.py      # project pipelines file
      settings.py       # project settings file
      spiders/          # a directory where you'll later put your spiders
          __init__.py
          ...
    {% endcodeblock %}
- 定义抓取目标（Item）
    - 本实例暂不使用
- 定义自己的爬虫(Spiders)
    1. 在spiders文件夹下新建spider.py文件
    2. 定义一个继承scrapy.Spider的BlogSpider类
    3. 定义BlogSpider类的name(名称)，start_urls(抓取网页url)和parse方法(对抓到的页面html的处理)
    {% codeblock lang:python %}
from scrapy import Spider
from scrapy.selector import Selector
import sys
sys.stdout=open('C:\out.txt','w') #将打印信息输出在相应的位置下

class BlogSpider(Spider):
    name = "blogspider"
    allowed_domains = ["github.io"]
    start_urls = [
    "http://nichijouyc.github.io/blog/archives/&quot;
    ]

    def parse(self, response):
        print 'Blog List:'
        sel = Selector(response)
        blogs = sel.xpath('//article/div/article/h1/a/text()').extract()
        for blog in blogs:
            print blog.encode('gb2312')
    {% endcodeblock%}
- 开始抓取
    - 在tutorial目录下执行`scrapy crawl blogspider`，在c盘的`out.txt`文件中出现本博客的博文列表，抓取成功。
