---
title: 一次python图片爬虫经历
date: 2018-04-11 17:02:56
tags: python
---

# 一次python图片爬虫经历
## 前言
听说python能干很多事情，比如写网络爬虫，于是就去看了些相关资料，python语法简单，库太多,随便想要什么功能的库都找得到,简直编程界的哆啦A梦；

本文是在mac环境下试验的，mac都预装了python2.x,然后本人又装了python3.x的版本了，所以执行文件时，我将会是用python3 xx.py来执行

首先要有点熟悉requests和BeautifulSoup的用法
BeautifulSoup可以去看一下其[开发文档](https://www.crummy.com/software/BeautifulSoup/bs4/doc.zh/)或者一些[BeautifulSoup入门博客](http://www.cnblogs.com/yupeng/p/3362031.html)
requests比urllib2(系统自带)简洁很多，可以去[官方网站](http://docs.python-requests.org/en/master/)了解，因为最新的文档是英文的，所有可以去看旧一点的[中文文档](http://requests-docs-cn.readthedocs.io/zh_CN/latest/)，基本教程没有什么差别

怎么安装库呢？python界也有个类似于我们iOS开发里cocoapods的东西,这个东西叫做pip.
由于我用的是python3.x的版本，所以我安装库同样会用pip3命令
![](/img/15234329959897.jpg)
![](/img/15234330148992.jpg)
这里你可以查看你安装了哪些库 用命令:`pip3 list`
![](/img/15234330820952.jpg)

## 方法
利用requests来爬取源码，BeautifulSoup来解析出图片地址，然后进行下载
目标网站:[http://www.jdlingyu.fun/](http://www.jdlingyu.fun/) 很多漂亮mm的网站

### 网页分享
获取了网页源代码了，里面都是html+css什么的,你想要从乱七八糟的一堆html里面找到正确的图片链接那可不是件简单的事，要具敏锐的洞察力
![](/img/15234335099327.jpg)
这里通过mac chrome的审查元素分析，图片入口url的规律:
被class为pin-coat的div给包裹着的a标签的href属性里的值就是入口url
当然它只是一个小图，这个不是很感兴趣，通过点击它，进入第二层，观看高清图，这里的图片才是我们想要的；
于是思路就是：
先得到第一层界面进入第二层界面的入口url（也就是那些小图），然后再得到第二层的高清图片的url，在进行下载就好了
![](/img/15234339689562.jpg)
在第二层，通过查找<img 标签 可以看出规律：
高清图为class为main-body的div中的a标签的href属性的值，所以只要得到这个url就可以了。
不多说，上代码： 注:(不生产代码，只是搬运工)

```python
#!/usr/bin/env python3
#coding:utf-8
import re
import requests
import bs4
if __name__ == "__main__":
    root_url = "http://www.jdlingyu.fun/page/"
    #为下载的照片命名，看有多少张绅士图
    k = 0
    for i in range(1,5+1):
        url = root_url + str(i) + "/"
        r = requests.get(url)
        #为了避免乱码
        html_cont_1 = r.content.decode('utf-8')
        soup_div_in = bs4.BeautifulSoup(html_cont_1,'html.parser')
        div_in = soup_div_in.find_all('div',class_='pin-coat')
        #BeautifulSoup返回的对象的类型不是String，而是数组，为了让BeautifulSoup进行二次索引，将其转换为String类型
        div_in_string = ''
        for x in div_in:
            div_in_string = div_in_string + str(x)
        #div_in_string里的值要为string类型    
        soup_link_in = bs4.BeautifulSoup(div_in_string,'html.parser')
        link_in = soup_link_in.find_all('a',href=re.compile(r"http://www.jdlingyu.fun/\d"))
        for link in link_in:
            url_in = link['href']
            r_in = requests.get(url_in)
            html_cont_2 = r_in.content.decode('utf-8')
            soup_div = bs4.BeautifulSoup(html_cont_2,'html.parser')
            div = soup_div.find_all('div',class_='main-body')
            div_string = ''
            for x in div:
                div_string = div_string + str(x)
            soup_link = bs4.BeautifulSoup(div_string,'html.parser')
            link_img = soup_link.find_all('a')
            for link in link_img:
                url_img = link['href']
                print(url_img)
                img_get = requests.get(url_img)
                #打开文件进行保存
                with open('img/'+str(k)+'.jpg','wb') as f:
                    f.write(img_get.content)
                k = k+1
```
运行这个代码要在代码的相同目录下创建存放图片的文件夹，不然会报没有该文件的错误
![](/img/15234342611180.jpg)

最终结果就是down了一大波图片，太多了，代码里的循环数据还可以改小点，没必要去down这么多
![](/img/15234344480822.jpg)


## 源码分析
python的语法简单，凡是#打头的就是python里面的注释语句类似于oc里的//.
这里也只是说明了下是python3环境 然后编码是utf-8

然后import了三个库,分别是re,requests,和bs4库.
re库是用来做正则表达式的，requests用来做发请求下load网页和下载的，bs4即beautifulsoup4用来解析html标签啥的

```
html_cont_1 = r.content.decode('utf-8')
soup_div_in = bs4.BeautifulSoup(html_cont_1,'html.parser')
div_in = soup_div_in.find_all('div',class_='pin-coat')
```
上面代码的意思就是通过获取网页源代码寻找html中所有div标签,并且这个div标签有个属性class,class的值是pin-coat.注意这个find_all函数,返回的是一个数组,所以我们div_in这个变量实际上是一个数组.

然后定义了一个div_in_string变量来将数组里的所有标签字串连接起来，然后再看代码

```
 soup_link_in = bs4.BeautifulSoup(div_in_string,'html.parser')
 link_in = soup_link_in.find_all('a',href=re.compile(r"http://www.jdlingyu.fun/\d"))
```
这里用了个正则，将所有的a标签，然后属性为href后带有http://www.jdlingyu.fun/数字的元素找出来，同样是一个数组

```
for link in link_in:
url_in = link['href']
r_in = requests.get(url_in)
html_cont_2 = r_in.content.decode('utf-8')
soup_div = bs4.BeautifulSoup(html_cont_2,'html.parser')
div = soup_div.find_all('div',class_='main-body')
```
接着遍历这个数组，首先得到我们高清图的网址，然后同样通过requests函数得到网页源码，然后编码，用beautifulsoup解析网页源码，找到所有属性为class值为main-body的div标签，得到一个数组；

```
for x in div:
div_string = div_string + str(x)
soup_link = bs4.BeautifulSoup(div_string,'html.parser')
link_img = soup_link.find_all('a')
```

然后同样是将上面数组里的所有标签连接起来，然后找到所有的a标签的元素，
同样是一个数组


```
for link in link_img:
url_img = link['href']
print(url_img)
img_get = requests.get(url_img)
with open('img/'+str(k)+'.jpg','wb') as f:
f.write(img_get.content)
k = k+1
```
遍历上面数组，然后数组里元素的href标签就能获取到最终我们要下载的图片的链接，然后下载图片，保存到img目录下；

记一次没有学过Python的我的爬虫经历。



