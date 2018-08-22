---
title: html解析器
date: 2018-01-03 19:14:52
tags: [html,spider]
categories: [spider]
---

因为公司的业务变更,最近让我搞起了python爬虫,现在是下班时间,所以把最近的东西整理整理.先来总结一下爬虫的html解析部分,这块也比较简单.

python爬虫的html解析器我目前使用了lxml和BeautifulSoup两种,接下来讲讲他们的使用方法.

### **lxml**

1. **lxml安装** 

    首先是安装,默认已经有了python环境和已安装pip,安装lxml的命令.
    ```
    $ pip install lxml
    ```

<!--  more -->

2. **lxml使用** 

    安装好以后,就可以使用lxml来进行xpath解析了,还不知道xpath??传送们:[xpath介绍](http://www.runoob.com/xpath/xpath-tutorial.html)

    2.1 **lxml读取文件**

    使用etree.parse('filename')来读取文件,代码如下:

    ```python
    from lxml import etree
    selector = etree.parse('demo.html')
    ```

    2.2 **lxml 读取html文本**

    ```python
        html = '''
        <html lang="en">
        <head>
            <meta charset="UTF-8"/>
            <title>Html parse demo</title>
        </head>
        <body>
        <div class="main">
            <div class="first">
                <img src="http://www.sinaimg.cn/dy/slidenews/21_img/2017_27/41065_5741488_547160.jpg" width="200px" height="200px" alt="美女图片"/>
                <p>这是文字</p>
            </div>
            <ul class="first">
                <li><a href="http://www.baidu.com">百度</a></li>
                <li><a href="http://www.zhihu.com">知乎</a></li>
                <li><a href="http://www.google.com">google</a></li>
                <li><a href="http://www.qq.com">QQ</a></li>
            </ul>
        </div>

        </body>
        </html>
        '''
    html_selector = etree.HTML(html)
    print_element(html_selector)
    
    ```

    2.3 **获取class为main的div标签**

    ```python
        main_div = selector.xpath('//div[@class="main"]')
        print_element(main_div[0])
    ```

    输出结果

    ```html
        <div class="main">
            <div class="first">
                <img src="http://www.sinaimg.cn/dy/slidenews/21_img/2017_27/41065_5741488_547160.jpg" width="200px" height="200px" alt="&#32654;&#22899;&#22270;&#29255;"/>
                <p>&#36825;&#26159;&#25991;&#23383;</p>
            </div>
            <ul class="first">
                <li><a href="http://www.baidu.com">&#30334;&#24230;</a></li>
                <li><a href="http://www.zhihu.com">&#30693;&#20046;</a></li>
                <li><a href="http://www.google.com">google</a></li>
                <li><a href="http://www.qq.com">QQ</a></li>
            </ul>
        </div>

    ```

    2.4 **读取main_div中的div/p中的文字信息**

    ```python
        text = main_div[0].findtext('div/p')
        print(text)
    ```

    > 查找element中的元素使用find(),查找某个标签中的文字使用findtext(),查找element中某个元素的所有标签findall()

    输出结果

    ```
        这是文字
    ```

    2.5 **获取main_div中的所有li标签中a标签中的href属性**

    ```python
        lis = main_div[0].findall('ul/li')
        for li in lis:
            # print_element(li)
            link = li.find('a')
            # 获取a标签中href的属性
            url = link.get('href')
            print(url)
    ```

    输出结果:

    ```
        http://www.baidu.com
        http://www.zhihu.com
        http://www.google.com
        http://www.qq.com
    ```

    2.6 **在循环中获取li的a标签的文字**

    ```python
        link_text = link.xpath('string(.)')
        print(link_text)
    ```

    输出结果:

    ```
        百度
        知乎
        google
        QQ
    ```

    2.7 **lxml xpath解析的完整例子**

    ```python
        from lxml import etree
        def print_element(element):
            string = etree.tostring(element,pretty_print=True).decode('utf-8')
            print(string)
        # 打开html文件
        selector = etree.parse('demo.html')
        # print_element(selector)
        # 获取class为main的div标签
        main_div = selector.xpath('//div[@class="main"]')
        print_element(main_div[0])
        # 读取main_div中的div/p中的文字信息
        text = main_div[0].findtext('div/p')
        print(text)
        # 获取main_div中的所有li标签
        lis = main_div[0].findall('ul/li')
        for li in lis:
            # print_element(li)
            link = li.find('a')
            # 获取a标签中href的属性
            url = link.get('href')
            print(url)
    ```
    
   

### **BeautifulSoup**

Beautiful Soup简介:

> Beautiful Soup 是一个可以从HTML或XML文件中提取数据的Python库.它能够通过你喜欢的转换器实现惯用的文档导航,查找,修改文档的方式.Beautiful Soup会帮你节省数小时甚至数天的工作时间.

1. **BeautifulSoup安装**
    ```
    $ pip install beautifulsoup4
    ```
2. **BeautifulSoup使用**

    2.1 **创建beautifulsoup对象**

    ```python
        soup = BeautifulSoup(html,'lxml')
    ```

    2.2 **打开本地html文件**

    ```python
        soup = BeautifulSoup(open('demo.html'),'lxml')
    ```

    2.3 **beautifulSoup格式化输出**

    ```python
        print(soup.prettify())
    ```

    2.4 **查找class为sister的a标签**

    ```python
        # 方式1
        links = soup.find_all('a',{'class':'sister'})
        # 方式2
        links = soup.find_all('a',class_='sister')

    ```

    2.5 **获取a标签中的href属性**

    ```python
        a = link.get('href')
    ```

    2.6 **获取a标签中的文字**

    ```python
        print(link.get_text())
    ```

    2.7 **完整的例子**

    ```python
        from bs4 import BeautifulSoup

        html = '''
            <html>
            <head>
            <title>
            The Dormouse's story
            </title>
            </head>
            <body>
            <p class="title">
            <b>
                The Dormouse's story
            </b>
            <span class="first second">
                Hello World
            </span>
            </p>
            <div class="test">
            <!-- 这是注释  -->
            </div>
            <p class="story">
            Once upon a time there were three little sisters; and their names were
            <a class="sister" href="http://example.com/elsie" id="link1">
                Elsie
            </a>
            ,
            <a class="sister" href="http://example.com/lacie" id="link2">
                Lacie
            </a>
            and
            <a class="sister" href="http://example.com/tillie" id="link2">
                Tillie
            </a>
            ; and they lived at the bottom of a well.
            </p>
            <p class="story">
            ...
            </p>
            </body>
            </html>
        '''

        # soup = BeautifulSoup(open('demo.html'),'lxml')
        # print(soup)

        soup = BeautifulSoup(html,'lxml')
        # print(soup)

        # # 查找class为sister的a标签
        # links = soup.find_all('a',{'class':'sister'})
        # links = soup.find_all('a',class_='sister')
        # # 获取a标签中的href属性
        # for link in links:
        #     # print(link)
        #     a = link.get('href')
        #     print(a)
        #     print(link.get_text())

        # span = soup.find('span',{'class':'first'})
        # print(span)

        # print(soup.a.string)

        # print(soup.prettify())

    ```

以上是基于lxml,BeautifulSoup最基本的解析html步骤,掌握以上就可解析大部分网页了,以后有补充的再加进来.



参考资料:
[xpath介绍](http://www.runoob.com/xpath/xpath-tutorial.html)
[Python爬虫利器三之Xpath语法与lxml库的用法](https://cuiqingcai.com/2621.html)
[lxml官方网站](http://lxml.de/)
[BeautifulSoup官网](https://www.crummy.com/software/BeautifulSoup/bs4/doc.zh/)
[Python爬虫利器二之Beautiful Soup的用法](https://cuiqingcai.com/1319.html)

