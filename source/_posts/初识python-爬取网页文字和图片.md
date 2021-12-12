---
title: 初识python-爬取网页文字和图片
author: 不好听
img: /2018/12/08/chu-shi-python-pa-qu-wang-ye-wen-zi-he-tu-pian/images/image1.jpg
categories: python
tags:
  - python
  - 爬虫
date: 2018-12-08 19:22:58
---

学习了一下用python爬取网络上的文字和图片，在此记录一下。  

### 1.下载“<font color=#0099ff face="微软雅黑">[笔趣看](https://www.biqukan.com/)</font>”网页上的小说  
代码如下：  
{% codeblock %}
# __*__ coding:UTF-8 __*__

import requests
from bs4 import BeautifulSoup

"""
下载 笔趣看 小说
"""


class DownloadBook(object):

    def __init__(self):
        self.server = 'https://www.biqukan.com'
        self.target = 'https://www.biqukan.com/20_20289/' 	# 书目在网站的地址
        self.name = '调教大宋.txt'  # 书名
        self.count = 0     # 章节数
        self.names = []     # 章节名
        self.urls = []      # 章节链接

    """
    获取所有章节的下载链接
    """
    def get_all_down_url(self):
        req = requests.get(url=self.target)
        bf = BeautifulSoup(req.text)
        texts = bf.find_all('div', class_='listmain')
        bf2 = BeautifulSoup(str(texts[0]))
        names = bf2.find_all('a')
        real = names[12: len(names)-7]
        self.count = len(real)
        for name in real:
            self.names.append(name.string)
            self.urls.append(self.server + name.get('href'))

    """
    获取章节内容
    """
    def get_every_content(self, target):
        req = requests.get(url=target)
        bf = BeautifulSoup(req.text)
        texts = bf.find_all('div', class_='showtxt')
        if len(texts) > 0 and texts[0].text:
            return texts[0].text.replace('\xa0' * 8, '\n')
        else:
            return ''

    """
    写入文件
    """
    def writer(self, path, name, text):
        with open(path, 'a', encoding='utf-8') as f:
            f.write(name + '\n')
            f.writelines(text)
            f.write('\n\n')


if __name__ == '__main__':
    obj = DownloadBook()
    obj.get_all_down_url()
    print('%s开始下载：', obj.name)
    for i in range(obj.count):
        obj.writer(obj.name, obj.names[i], obj.get_every_content(obj.urls[i]))
        print('\r已下载：%.3f %%' % float(i / obj.count * 100), end='')
    print('%s下载完成。', obj.name)
	
{% endcodeblock %}  
结果如下：  
![结果](images/11.png "结果")  

<br><br>

### 2.爬取“<font color=#0099ff face="微软雅黑">[彼岸桌面](http://www.netbian.com/)</font>”网站上的壁纸  
代码如下： 
{% codeblock %} 
# __*__ coding:UTF-8 __*__

import requests
import os
import time

from bs4 import BeautifulSoup
from contextlib import closing

"""
下载 彼岸桌面 壁纸
"""


class DownloadPic(object):

    def __init__(self):
        self.server = 'http://www.netbian.com/'
        self.target = 'http://www.netbian.com/'
        self.img_server = 'http://img.netbian.com/'
        self.urls = []      # 图片列表解析的地址
        self.big_urls = []     # 图片的大图地址
        self.real_urls = []     # 图片下载地址
        self.names = []     # 图片名

    """
    获取每页url集合
    """
    def get_url_list(self):
        req = requests.get(url=self.target)
        bf = BeautifulSoup(req.text)
        texts = bf.find_all('div', class_='list')
        if len(texts) <= 0:
            return
        else:
            bf2 = BeautifulSoup(str(texts[0]))
            urls = bf2.find_all('a')
            for k in range(len(urls)):
                if k % 2 == 0 and k != 4:       # 每个url重复，且第三个图片（即i=(4,5)时是广告）
                    self.urls.append(self.server + urls[k].get('href'))

    """
    获取每张图片的 大图浏览页的地址
    """
    def get_big_url(self):
        for url in self.urls:
            req = requests.get(url=url)
            bf = BeautifulSoup(req.text)
            texts = bf.find_all('div', class_='pic')
            bf2 = BeautifulSoup(str(texts[0]))
            urls = bf2.find_all('a')
            self.big_urls.append(self.server + urls[0].get('href'))

    """
    获取图片的下载地址
    """
    def get_real_urls(self):
        for url in self.big_urls:
            req = requests.get(url=url)
            bf = BeautifulSoup(req.text)
            texts = bf.find_all('td', align='left')
            bf2 = BeautifulSoup(str(texts[0]))
            urls = bf2.find_all('a')
            self.real_urls.append(urls[0].get('href'))
            self.names.append(urls[0].get('title'))

    """
    下载图片
    """
    def download(self):
        path = os.path.abspath('.')
        path2 = os.path.join(path, 'pic')
        if os.path.isdir(path2):
            pass
        else:
            os.mkdir(path2)
        for j in range(len(self.real_urls)):
            time.sleep(1)
            pu = os.path.join(path2, self.names[j]+'.jpg')
            print(self.real_urls[j])
            with closing(requests.get(url=self.real_urls[j], stream=True, verify=False)) as r:
                with open(pu, 'ab+') as f:
                    for chunk in r.iter_content(chunk_size=1024):
                        if chunk:
                            f.write(chunk)
                            f.flush()


if __name__ == '__main__':
    for i in range(3):  # 爬取3页
        obj = DownloadPic()
        if i > 0:
            index = i+1
            obj.target = obj.target + 'index_' + str(index) + '.htm'
        else:
            pass
        obj.get_url_list()
        obj.get_big_url()
        obj.get_real_urls()
        obj.download()
		
{% endcodeblock %}   
结果如下：  
![结果](images/12.png "结果")  

<br>

问题：  
  爬取的图片无法打开，经查看图片显示的大小仅1kb，无法打开；经查找发现是获取的图片路径有误，响应为403错误，程序不能正常解析导致。

Tip: 以上内容仅用于学习！