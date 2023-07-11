---
title: bs4基础语法
cover: 'https://pic.cwwwxl.top/img/t01ba63556436f505a3.jpg'
tags: 爬虫
categories: python
top_img: 'https://pic.cwwwxl.top/img/t01ba63556436f505a3.jpg'
abbrlink: 8a8d0b38
---
### 1.1 .BeautifulSoup  

```python
1.BeautifulSoup简称:
 bs4
2.什么是BeatifulSoup?
Beautifulsoup，和lxml一样，是一个html的解析器，主要功能也是解析和提取数据
3.优缺点?
缺点：效率没有lxml的效率高
优点：接口设计人性化，使用方便
```

##### 1.1.2安装以及创建

```python
1.安装
pip install bs4
2.导入
from bs4 import BeautifulSoup
3.创建对象
服务器响应的文件生成对象     
soup = BeautifulSoup(response.read().decode()，'lxml')
本地文件生成对象
soup = BeautifulSoup(open('1.html')，'lxml')
注意：默认打开文件的编码格式gbk所以需要指定打开编码格式
```

##### 1.1.3 节点查询

```python
1.根据标签名查找节点
soup.a 【注】只能找到第一个a
soup.a.name
soup.a.attrs
2.函数
(1).find(返回一个对象）
	find('a')：只找到第一个a标签
	find('a', title='名字'）
	find('a',class_='名字'）
（2).find_all(返回一个列表）
    find_all('a') 查找到所有的a
    find_all(['a','span']) 返回所有的a和span
    find_all('a', limit=2） 只找前两个a
(3).select(根据选择器得到节点对象）【推荐】
1.element
	 eg:p
2..class
	eg:.firstname
3.#id
	 eg:#firstname
4.属性选择器
[attribute]
	eg:li = soup.select('li[class]'）
[attribute=value]
	eg:li = soup.select('li[class="hengheng1"]'）
                    5.层级选择器
element element
div p
element>element
div>p
element,element
div,p
	eg：soup = soup.select('a,span')
 
```

##### 1.1.4节点信息

```python
（1）.获取节点内容：适用于标签中嵌套标签的结构
obj.string
obj.get_text(）【推荐】
（2）.节点的属性
tag.name 获取标签名
	eg:tag = find('li)
print(tag.name）
tag.attrs将属性值作为一个字典返回
（3）.获取节点属性
obj.attrs.get('title')【常用】
obj.get('title')
obj['title']
```

```python
from bs4 import BeautifulSoup

# 默认打开文件是gbk 需要指定编码
soup = BeautifulSoup(open('static/test.html', encoding='utf-8'), 'lxml')
# 根据标签内容找数据，找到的是第一个数据
# print(soup.table)
# 返回的标签的属性
# print(soup.a.attrs)
# bs4的一些函数方法
# (1) find
# 返回第一个符合条件的数据,根据属性值查找标签
# print(soup.find('table',width='500'))

# (2) find_all 返回列表
# print(soup.find_all('td'))

#如果要多个标签的数据需要用[]
# print(soup.find_all(['a','td']))
#limit 指定数据个数
# print(soup.find_all('td',limit=10))
# (3) select
#返回多个数据
# print(soup.select('td'))
#通过类选择器 class
# print(soup.select('.a1'))
# # 通过id
# print(soup.select('#a1'))
# 找到table中有id的数据
# print(soup.select('table[id]'))
#层级选择器 后代选择器
# print( soup.select('table tr td'))
#子代选择器
obj= soup.select('table > tr > td')[0]
print(obj.string)
print(obj.get_text())
#获取节点内容


```

