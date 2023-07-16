---
title: Python爬虫
date: 2019-02-11 14:23:37
categories: Python学习
tags: 
- Python
- 爬虫
---
网络爬虫之规则
======
<!--more-->

Requests库入门
------
### pip3与Requests库的安装
pip是python的包管理工具，pip和pip3版本不同,建议使用pip3.
pip3的安装参考的下面的链接:
[https://blog.csdn.net/qq_30163461/article/details/80396258](https://blog.csdn.net/qq_30163461/article/details/80396258) 
Requests库的安装只需要下面一条命令:

```
pip3 install requests
```

至于Requests库的使用,只需要在代码最开始加上下面这行代码就可以了.
```
import requests
```

### Requests库的简单介绍
Requests库的常用方法如下:
![](./Python爬虫/Requests库的7个主要方法.png)
http协议中对资源的操作方法如下:
![](./Python爬虫/方法.png)
下面一一介绍上面的这七个方法:
**requests.request(method,url,\*\*kwargs)**
构造并发送一个**Request对象**，返回一个**Response对象**，支撑各方法的基础方法
该方法的各参数含义如下:

+ method: 请求方式,对应get/put/post等7种
+ url: 拟获取页面的url链接
+ \*\*kwargs: 控制访问的参数,共13个

![](./Python爬虫/method.png)
![](./Python爬虫/kwargs.png)
![](./Python爬虫/kwargs续.png)
下面是一些控制访问参数的实例:

```py
#params参数:字典或字节序列,作为参数增加到url中
>>> kv = {'key1': 'value1', 'key2': 'value2'}
>>> r = requests.request('GET', 'http://python123.io/ws', params=kv)
>>> print(r.url)
http://python123.io/ws?key1=value1&key2=value2

#data参数:字典、字节序列或文件对象,作为Request的内容
>>> kv = {'key1': 'value1', 'key2': 'value2'}
>>> r = requests.request('POST', 'http://python123.io/ws', data=kv)
>>> body = '主体内容'
>>> r = requests.request('POST', 'http://python123.io/ws', data=body)

#json参数:JSON格式的数据,作为Request的内容
>>> kv = {'key1': 'value1'}
>>> r = requests.request('POST', 'http://python123.io/ws', json=kv)

#headers : 字典,HTTP定制头
>>> hd = {'user‐agent': 'Chrome/10'}
>>> r = requests.request('POST', 'http://python123.io/ws', headers=hd)

#files   : 字典类型,传输文件
>>> fs = {'file': open('data.xls', 'rb')}
>>> r = requests.request('POST', 'http://python123.io/ws', files=fs)

#timeout : 设定超时时间,秒为单位
>>> r = requests.request('GET', 'http://www.baidu.com', timeout=10)

#proxies : 字典类型,设定访问代理服务器,可以增加登录认证
>>> pxs = { 'http': 'http://user:pass@10.10.10.1:1234'
			'https': 'https://10.10.10.1:4321'}
>>> r = requests.request('GET', 'http://www.baidu.com', proxies=pxs)
```
其他方法如下:

```python
requests.get(url, params=None, **kwargs)
requests.head(url, **kwargs)
requests.post(url, data=None, json=None, **kwargs)
requests.put(url, data=None, **kwargs)
requests.patch(url, data=None, **kwargs)
requests.delete(url, **kwargs)
```

这些方法都是在request方法的基础上实现的,kwargs参数也与request方法中的用法相同,只是要去掉在方法中所声明的参数.
其中get方法最为常用,它会构造一个向服务器请求资源的Request对象,返回一个包含服务器资源的Response对象,下面看一下Response对象.

**Response对象包含服务器返回的所有信息,也包含请求的Request信息**.
Response对象有如下常用属性:
![](./Python爬虫/Response对象属性.png)
其中,r.encoding属性的工作原理如下:r.encoding的值等于header中的charset的值,如果header中不存在charset,则认为编码为ISO-8859-1,r.text根据r.encoding显示网页内容.
r.apparent_encoding则是根据网页内容分析出的编码方式,可以看做是r.encoding的备选.

在代码执行过程中,可能会发生各种异常,Requests库中的异常如下:
![](./Python爬虫/Requests库异常.png)
在Response对象中,有一个常用的与异常有关的方法:

```
r.raise_for_status()
```

r.raise_for_status()在方法内部判断r.status_code是否等于200,如果不是200,产生异常 requests.HTTPError,不需要增加额外的if语句,该语句便于利用try-except进行异常处理

requests库使用的通用代码框架如下:
```py
try:
    r = requests.get(url,timeout=30)
    r.raise_for_status()	#如果状态不是200,引发HTTPError异常
    r.encoding = r.apparent_encoding
    return r.text
except:
    return "产生异常"
```

网络爬虫的"盗亦有道"
------
网站对网络爬虫的限制有以下两种方式:

+ 判断User-Agent进行限制
	检查来访HTTP协议头的User-Agent域,只响应浏览器或友好爬虫的访问
+ 发布公告:Robots协议
	告知所有爬虫网站的爬取策略,要求爬虫遵守

下面介绍一下Robots协议,Robots协议全名为Robots Exclusion Standard,为网络爬虫排除标准.它的作用是告知网络爬虫网站的哪些页面可以抓取,哪些不行.其形式为**网站根目录下的robots.txt文件**

Robots协议的基本语法如下:

```
#表示注释,*代表所有,/代表根目录
User-agent: * 
Disallow: / 
```

下面看一下Robots协议的实例,这是京东网站下的robots.txt内容:

```
User‐agent: * 
Disallow: /?* 
Disallow: /pop/*.html 
Disallow: /pinpai/*.html?* 
User‐agent: EtaoSpider
Disallow: / 
User‐agent: HuihuiSpider
Disallow: / 
User‐agent: GwdangSpider
Disallow: / 
User‐agent: WochachaSpider
Disallow: /
```

网络爬虫之提取
======
Beautiful Soup库入门
------
首先安装Beautiful Soup库,只要执行以下命令就可以了:

	pip3 install beautifulsoup4

### Beautiful Soup库的基本元素
Beautiful Soup库是解析,遍历,维护"标签树"的功能库.
html的树型结构跟java进阶中的DOM示例几乎一样,两个标签之间的换行也是一个节点,为NavigableString类型;并且标签中的文本也是一个NavigableString类型的节点.
Beautiful Soup库,也叫beautifulsoup4或bs4,一般有两种引用方式:
```py
from bs4 import BeautifulSoup
import bs4
```
常用第一种方式,即主要使用BeautifulSoup类.**BeautifulSoup类**对应一个HTML/XML文档的全部内容.

Beautiful Soup库有以下四种解析器:
![](./Python爬虫/bs4解析器.png)
其中的mk可以是HTML/XML字符串或HTML/XML文档对象.

BeautifulSoup类的基本元素
![](./Python爬虫/BeautifulSoup类的基本元素.png)
现在有一名为demo的html字符串内容如下:
```html
<html>
 <head>
  <title>
   This is a python demo page
  </title>
 </head>
 <body>
  <p class="title">
   <b>
    The demo python introduces several python courses.
   </b>
  </p>
  <p class="course">
   Python is a wonderful general-purpose programming language. You can learn Python from novice to professional by tracking the following courses:
   <a class="py1" href="http://www.icourse163.org/course/BIT-268001" id="link1">
    Basic Python
   </a>
   and
   <a class="py2" href="http://www.icourse163.org/course/BIT-1001870001" id="link2">
    Advanced Python
   </a>
   .
  </p>
 </body>
</html>

```
任何存在与HTML语法中的标签都可以用soup.&lt;tag&gt;访问获得,当HTML文档中存在多个相同&lt;tag&gt;对应内容时,soup.&lt;tag&gt;返回第一个
示例如下:
```py
>>> from bs4 import BeautifulSoup
>>> soup = BeautifulSoup(demo,"html.parser")
>>> soup.title	#文档中的标签
<title>This is a python demo page</title>
>>> tag = soup.a
>>> tag
<a class="py1" href="http://www.icourse163.org/course/BIT-268001" id="link1">Basic Python</a>

#每个<tag>都有自己的名字,通过<tag>.name获取,字符串类型
>>> soup.a.name
'a'
>>> soup.a.parent.name
'p'
>>> soup.a.parent.parent.name
'body'

#一个<tag>可以有0或多个属性,字典类型
>>> tag = soup.a
>>> tag.attrs
{'href': 'http://www.icourse163.org/course/BIT-268001', 'class': ['py1'], 'id': 'link1'}
>>> tag.attrs['class']
['py1']
>>> tag.attrs['href']
'http://www.icourse163.org/course/BIT-268001'
>>> type(tag.attrs)
<class 'dict'>
>>> type(tag)
<class 'bs4.element.Tag'>

#NavigableString可以跨越多个层次
>>> soup.a
<a class="py1" href="http://www.icourse163.org/course/BIT-268001" id="link1">Basic Python</a>
>>> soup.a.string
'Basic Python'
>>> soup.p
<p class="title"><b>The demo python introduces several python courses.</b></p>
>>> soup.p.string
'The demo python introduces several python courses.'
>>> type(soup.p.string)
<class 'bs4.element.NavigableString'>

#标签内字符串的注释部分,一种特殊的Comment类型
>>> newsoup = BeautifulSoup("<b><!--This is a comment--></b><p>This is not a comment</p>","html.parser")
>>> newsoup.b.string
'This is a comment'
>>> type(newsoup.b.string)
<class 'bs4.element.Comment'>
>>> type(newsoup.p.string)
<class 'bs4.element.NavigableString'>
```
![](./Python爬虫/基本元素图解.png)
在BeautifulSoup对象中,还有一个.text属性,跟.string属性很相似,但还是有一些区别,例如有以下html文本:
```html
<td>some text</td>
<td></td>
<td><p>more text</p></td>
<td>even <p>more text</p></td>
```
.string会有如下的返回结果:
```
some text
None
more text
None
```
.text会返回如下结果
```
some text

more text
even more text
```
他们的主要区别在第二行与第四行.
第二行指定标签内没有文本时,.string返回None,.text返回为空
第四行指定标签内的文本数量大于等于2,.string不知道获取哪一个,所以返回为空.而.text返回的是两段文本的拼接.

### 基于bs4库的HTML内容遍历方法
bs4库的HTML内容遍历方法有上行遍历,下行遍历,平行遍历,如下图:
![](./Python爬虫/HTML基本格式.png)

![](./Python爬虫/下行遍历.png)
示例程序如下:
```py
#contents属性
>>> soup = BeautifulSoup(demo,"html.parser")
>>> soup.head
<head><title>This is a python demo page</title></head>
>>> soup.head.contents
[<title>This is a python demo page</title>]
>>> soup.body.contents	#contents会返回标签后面的换行,作为NavigableString类型
['\n', <p class="title"><b>The demo python introduces several python courses.</b></p>, '\n', <p class="course">Python is a wonderful general-purpose programming language. You can learn Python from novice to professional by tracking the following courses:
<a class="py1" href="http://www.icourse163.org/course/BIT-268001" id="link1">Basic Python</a> and <a class="py2" href="http://www.icourse163.org/course/BIT-1001870001" id="link2">Advanced Python</a>.</p>, '\n']

#遍历儿子节点
for child in soup.body.children:
    print(child)

#遍历子孙节点
for child in soup.body.descendants:
    print(child)
```
**注意,contents返回子节点时,也会返回标签后的换行(如果有的话),作为NavigableString类型的节点**,比如上面的示例程序,返回的列表中有3个换行,两个p标签节点.

![](./Python爬虫/上行遍历.png)
示例程序如下:
```py
>>> soup = BeautifulSoup(demo,"html.parser")
>>> soup.title.parent
<head><title>This is a python demo page</title></head>
>>> soup.html.parent	#html标签的父亲标签为整个文档
<html><head><title>This is a python demo page</title></head>
<body>
<p class="title"><b>The demo python introduces several python courses.</b></p>
<p class="course">Python is a wonderful general-purpose programming language. You can learn Python from novice to professional by tracking the following courses:
<a class="py1" href="http://www.icourse163.org/course/BIT-268001" id="link1">Basic Python</a> and <a class="py2" href="http://www.icourse163.org/course/BIT-1001870001" id="link2">Advanced Python</a>.</p>
</body></html>
>>> soup.parent
>>> print(soup.parent)	#BeautifulSoup对象的父亲标签为None
None

#遍历所有先辈节点,包括soup本身,所以要区别判断
>>> for parent in soup.a.parents:
...     if parent is None:
...         break
...     else:
...         print(parent.name)
... 
p
body
html
[document]
```
![](./Python爬虫/平行遍历.png)
示例程序如下:
```py
>>> soup = BeautifulSoup(demo,"html.parser")
>>> soup.a.next_sibling
' and '
>>> soup.a.next_sibling.next_sibling
<a class="py2" href="http://www.icourse163.org/course/BIT-1001870001" id="link2">Advanced Python</a>
>>> soup.a.previous_sibling
'Python is a wonderful general-purpose programming language. You can learn Python from novice to professional by tracking the following courses:\r\n'
>>> soup.a.previous_sibling.previous_sibling	#a标签前面没有兄弟节点了,所以返回值为空
>>> print(soup.a.previous_sibling.previous_sibling)
None
>>> soup.a.parent
<p class="course">Python is a wonderful general-purpose programming language. You can learn Python from novice to professional by tracking the following courses:
<a class="py1" href="http://www.icourse163.org/course/BIT-268001" id="link1">Basic Python</a> and <a class="py2" href="http://www.icourse163.org/course/BIT-1001870001" id="link2">Advanced Python</a>.</p>

#循环遍历后序节点
>>> for sibling in soup.a.next_siblings:
...     print(sibling)
... 
 and 
<a class="py2" href="http://www.icourse163.org/course/BIT-1001870001" id="link2">Advanced Python</a>
.

#循环遍历前序节点
>>> for sibling in soup.a.previous_siblings:
...     print(sibling)
... 
Python is a wonderful general-purpose programming language. You can learn Python from novice to professional by tracking the following courses:

```
![](./Python爬虫/平行遍历示意.png)
需要注意的是,平行遍历发生在**同一个父节点**下的各节点间

### 基于bs4库的HTML格式输出
bs4库中有一个.prettify()方法,.prettify()为HTML文本标签及其内容增加'\n',.prettify()也可用于标签,使用方法:&lt;tag&gt;.prettify().
示例程序如下:
```py
>>> soup
<html><head><title>This is a python demo page</title></head>
<body>
<p class="title"><b>The demo python introduces several python courses.</b></p>
<p class="course">Python is a wonderful general-purpose programming language. You can learn Python from novice to professional by tracking the following courses:
<a class="py1" href="http://www.icourse163.org/course/BIT-268001" id="link1">Basic Python</a> and <a class="py2" href="http://www.icourse163.org/course/BIT-1001870001" id="link2">Advanced Python</a>.</p>
</body></html>
>>> soup.prettify()
'<html>\n <head>\n  <title>\n   This is a python demo page\n  </title>\n </head>\n <body>\n  <p class="title">\n   <b>\n    The demo python introduces several python courses.\n   </b>\n  </p>\n  <p class="course">\n   Python is a wonderful general-purpose programming language. You can learn Python from novice to professional by tracking the following courses:\n   <a class="py1" href="http://www.icourse163.org/course/BIT-268001" id="link1">\n    Basic Python\n   </a>\n   and\n   <a class="py2" href="http://www.icourse163.org/course/BIT-1001870001" id="link2">\n    Advanced Python\n   </a>\n   .\n  </p>\n </body>\n</html>'
>>> print(soup.prettify())
<html>
 <head>
  <title>
   This is a python demo page
  </title>
 </head>
 <body>
  <p class="title">
   <b>
    The demo python introduces several python courses.
   </b>
  </p>
  <p class="course">
   Python is a wonderful general-purpose programming language. You can learn Python from novice to professional by tracking the following courses:
   <a class="py1" href="http://www.icourse163.org/course/BIT-268001" id="link1">
    Basic Python
   </a>
   and
   <a class="py2" href="http://www.icourse163.org/course/BIT-1001870001" id="link2">
    Advanced Python
   </a>
   .
  </p>
 </body>
</html>
>>> soup.a.prettify()
'<a class="py1" href="http://www.icourse163.org/course/BIT-268001" id="link1">\n Basic Python\n</a>\n'
```
bs4库将任何HTML输入都变成utf-8编码,Python 3.x默认支持编码是utf-8,解析无障碍

信息标记与提取方法
------
### 信息标记的三种形式
![](./Python爬虫/信息标记引子.png)
信息标记主要有三种形式:XML,JSON,YAML.
XML与JSON在这里不做介绍,YAML的介绍可以参考下面的两个链接.
[https://blog.csdn.net/lilun517735159/article/details/79230732](https://blog.csdn.net/lilun517735159/article/details/79230732) 
[http://www.ruanyifeng.com/blog/2016/07/yaml.html?f=t](http://www.ruanyifeng.com/blog/2016/07/yaml.html?f=t) 

### 基于bs4的HTML内容查找方法
bs4库中常用.find_all()方法查找内容:
![](./Python爬虫/find_all.png)
&lt;tag&gt;.find_all( name , attrs , recursive , string , \*\*kwargs )
.find_all() 方法搜索当前tag的所有tag子节点,并判断是否符合**过滤器(字符串,正则表达式,列表,方法或是True)**的条件,返回一个**列表类型**,存储查找的结果.
下面一一介绍该方法中的参数,下面的示例程序所使用的还是上面的html文本.

**name参数**
name参数可以查找所有名字为name的tag,字符串对象会被自动忽略掉.
示例程序如下:
```py
>>> soup.find_all("a")
[<a class="py1" href="http://www.icourse163.org/course/BIT-268001" id="link1">Basic Python</a>, <a class="py2" href="http://www.icourse163.org/course/BIT-1001870001" id="link2">Advanced Python</a>]
>>> soup.find_all(["a","b"])
[<b>The demo python introduces several python courses.</b>, <a class="py1" href="http://www.icourse163.org/course/BIT-268001" id="link1">Basic Python</a>, <a class="py2" href="http://www.icourse163.org/course/BIT-1001870001" id="link2">Advanced Python</a>]
>>> for tag in soup.find_all(True):	#查找所有的tag
...     print(tag.name)
... 
html
head
title
body
p
b
p
a
a
```
name参数的值可以是任一类型的**过滤器**:字符串,正则表达式,列表,方法或是True.**True可以匹配任何值**.

**keyword参数**
如果一个指定名字的参数不是搜索内置的参数名,搜索时会把该参数当作指定名字tag的属性来搜索.但是有些特殊的tag属性在搜索中不能使用,这时可以通过**attrs**参数定义一个**字典参数**来搜索包含特殊属性的tag.此外,还有一个特殊情况:class属性.由于class在python中是保留字,使用class做参数会导致语法错误,这个问题有两种解决方式:一种是使用**class_参数**搜索有指定类名的tag;另一种是使用attrs参数.
示例程序如下:
```py
>>> soup.find_all(id="link1")
[<a class="py1" href="http://www.icourse163.org/course/BIT-268001" id="link1">Basic Python</a>]
>>> soup.find_all("a","py1")	#这条语句作用与下面那条语句相同
[<a class="py1" href="http://www.icourse163.org/course/BIT-268001" id="link1">Basic Python</a>]
>>> soup.find_all(class_="py1")
[<a class="py1" href="http://www.icourse163.org/course/BIT-268001" id="link1">Basic Python</a>]
>>> soup.find_all(attrs={"class":"py1"})
[<a class="py1" href="http://www.icourse163.org/course/BIT-268001" id="link1">Basic Python</a>]
>>> soup.find_all(id="link1",class_="py1")	#使用多个指定名字的参数可以同时过滤tag的多个属性
[<a class="py1" href="http://www.icourse163.org/course/BIT-268001" id="link1">Basic Python</a>]
```

**string参数**
通过string 参数可以搜搜文档中的字符串内容.与name参数的可选值一样,string参数接受字符串 ,正则表达式 ,列表,True .还有一个**text参数**跟string参数功能相似.
示例代码如下:
```py
>>> soup.find_all(string="Basic Python")
['Basic Python']
>>> soup.find_all(text="Basic Python")
['Basic Python']
```

**limit 参数**
如果我们不需要全部结果,可以使用 limit 参数限制返回结果的数量.当搜索到的结果数量达到 limit 的限制时,就停止搜索返回结果.
示例代码如下:
```py
>>> soup.find_all("a")
[<a class="py1" href="http://www.icourse163.org/course/BIT-268001" id="link1">Basic Python</a>, <a class="py2" href="http://www.icourse163.org/course/BIT-1001870001" id="link2">Advanced Python</a>]
>>> soup.find_all("a",limit=1)
[<a class="py1" href="http://www.icourse163.org/course/BIT-268001" id="link1">Basic Python</a>]
```

**recursive参数**
调用tag的 find_all() 方法时,Beautiful Soup会检索当前tag的所有子孙节点,如果只想搜索tag的直接子节点,可以使用参数 recursive=False .

特别的:**&lt;tag&gt;(...)等价于&lt;tag&gt;.find_all(...)**
**soup(...)等价于soup.find_all(...)**

此外还有一些额外的扩展方法:
![](./Python爬虫/扩展方法.png)
find_all() 方法的返回结果是值包含一个元素的列表,而 find() 方法直接返回结果.find_all() 方法没有找到目标是返回空列表, find() 方法找不到目标时,返回 None .

另外,python的格式化字符串存在一个中文不能很好的对齐的问题,其原因是当中文字符宽度不够时,采用西文字符填充;而中西文字符占用的宽度不同,因此产生了中文字符的对齐问题.解决方法是采用中文字符的空格填充:chr(12288).
示例:
```py
print("{:^10}\t{:^6}\t{:^10}".format("排名","学校名称","总分"))	#中文字符空格填充前
print("{0:^10}\t{1:{3}^10}\t{2:^10}".format("排名","学校名称","总分",chr(12288)))	#中文字符空格填充后
```

[https://www.cnblogs.com/keye/p/7868059.html](https://www.cnblogs.com/keye/p/7868059.html) 
[https://beautifulsoup.readthedocs.io/zh_CN/v4.4.0/#recursive](https://beautifulsoup.readthedocs.io/zh_CN/v4.4.0/#recursive)

网络爬虫之实战
======
Re(正则表达式)库入门
------
正则表达式(regular expression,regex,RE)是用来简洁表达一组字符串的表达式.
下面是正则表达式的常用操作符:
![](./Python爬虫/正则表达式常用操作符1.png)
![](./Python爬虫/正则表达式常用操作符2.png)
对正则表达式,在这里不多做介绍,只介绍一下Re库.Re库是python的标准库,主要用于字符串匹配.调用方式如下:
```py
import re
```

正则表达式的表示一般使用raw string类型(原生字符串类型),表示为r'text',在普通字符串前加上一个字符r.例如:

	r'[1-9]\d{5}'
	r'\d{3}-\d{8}|\d{4}-\d{7}'

raw string中转义符\\只是一个普通的字符,不具有转义功能.
热库也可以使用string类型表示正则表达式,但更繁琐,例如:

	'[1-9]\\d{5}'
	'\\d{3}-\\d{8}|\\d{4}-\\d{7}'

因此,当正则表达式中包含转义符时,建议使用raw string.

### Re库的主要功能函数
![](./Python爬虫/re库主要功能函数.png)
下面对这几个函数进行介绍:

re.search函数:
![](./Python爬虫/search.png)
flags参数对应的几个常用的标记如下:
![](./Python爬虫/常用标记.png)
示例代码如下:
```py
>>> import re
>>> match = re.search(r"[1-9]\d{5}","BIT 100081")
>>> if match:
...     print(match.group(0))
... 
100081
```

re.match函数:
![](./Python爬虫/match.png)
示例代码如下:
```py
>>> import re
>>> match = re.match(r"[1-9]\d{5}","BIT 100081")
>>> print(match)	#由于字符串的开始位置无法匹配正则表达式,所以返回None
None
>>> match = re.match(r"[1-9]\d{5}","100081 BIT")
>>> if match:
...     match.group(0)
... 
'100081'
```

re.findall函数:
![](./Python爬虫/refindall.png)
示例代码如下:
```py
>>> import re
>>> ls = re.findall(r"[1-9]\d{5}","BIT100081 TSU100084")
>>> ls
['100081', '100084']
```

re.split函数:
![](./Python爬虫/resplit.png)
re.split函数会将正则表达式匹配结果作为分割点对字符串进行分割,并将分割的结果作为列表返回.
maxsplit参数默认为0,表示每个匹配项都分割
示例代码如下:
```py
>>> import re
>>> re.split(r'[1-9]\d{5}','BIT100081 TSU100084')
['BIT', ' TSU', '']
>>> re.split(r'[1-9]\d{5}','BIT100081 TSU100084',maxsplit=1)
['BIT', ' TSU100084']
```

re.finditer函数:
![](./Python爬虫/refinditer.png)
示例代码如下:
```py
>>> import re
>>> for m in re.finditer(r"[1-9]\d{5}","BIT100081 TSU100084"):
...     if m:
...         print(m.group(0))
... 
100081
100084
```

re.sub函数:
![](./Python爬虫/resub.png)
count参数默认为0,表示全部进行替换
示例代码如下:
```py
>>> import re
>>> re.sub(r'[1-9]\d{5}',':zipcode','BIT100081 TSU100084')
'BIT:zipcode TSU:zipcode'
>>> re.sub(r'[1-9]\d{5}',':zipcode','BIT100081 TSU100084',count=1)
'BIT:zipcode TSU100084'
```

除了上面所说的用法,re库还有另外一种等价用法,见下图:
![](./Python爬虫/re库等价用法.png)
可以使用re.compile函数将正则表达式的字符串形式编译成**Pattern对象**(正则表达式对象).
示例代码如下:
```py
>>> type(re.compile(r'[1-9]\d{5}'))
<class '_sre.SRE_Pattern'>
```
re.complie函数的详细用法见下图:
![](./Python爬虫/compile.png)
编译后产生的Pattern对象有着跟上面的函数相对应的方法,只是不需要pattern参数了.
![](./Python爬虫/pattern对象方法.png)

### Re库的Match对象
Match对象是一次匹配的结果,包含匹配的很多信息.
示例代码如下:
```py
>>> match = re.search(r'[1-9]\d{5}','BIT 100081')
>>> type(match)
<class '_sre.SRE_Match'>
```
Match对象中有如下常用属性:
![](./Python爬虫/match对象属性.png)
上图中有一个错误,.re代表的是匹配时使用的Pattern对象(正则表达式).

Match对象中有如下常用方法:
![](./Python爬虫/match对象方法.png)
.group函数用来得到正则表达式中的各分组(正则表达式用括号()分组)所匹配的字符串,这里的.group(0)和.group()都返回与正则表达式匹配的整体结果,而.group(1)则返回第一个分组所匹配的结果,.group(2)与.group(3)同理.
示例代码如下
```py
>>> match = re.search(r'(\d{3})(\w{3})','123abc')
>>> match.group()
'123abc'
>>> match.group(0)
'123abc'
>>> match.group(1)
'123'
>>> match.group(2)
'abc'
>>> match.group(3)	#没有第三个分组,引发错误
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
IndexError: no such group
```

对上面的Match对象的属性与方法的示例程序如下:
```py
>>> m = re.search(r'[1-9]\d{5}','BIT100081 TSU100084')
>>> m.string
'BIT100081 TSU100084'
>>> m.re
re.compile('[1-9]\\d{5}')
>>> m.pos
0
>>> m.endpos
19
>>> m.group(0)
'100081'
>>> m.start()
3
>>> m.end()
9
>>> m.span()
(3, 9)
```

在正则表达式中,有贪婪匹配和最小匹配,而Re库默认采用贪婪匹配,即输出匹配最长的子串.如果想要使用最小匹配,只需要在限定符后面加上一个?即可.最小匹配操作符如下:
![](./Python爬虫/最小匹配操作符.png)

[https://www.cnblogs.com/smallmars/p/6935269.html](https://www.cnblogs.com/smallmars/p/6935269.html) 
