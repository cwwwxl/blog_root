---
title: xpath基础语法
cover: 'https://pic.cwwwxl.top/img/t01a6261ec30093fe4b.jpg'
tags: 爬虫
categories: python
top_img: 'https://pic.cwwwxl.top/img/t01a6261ec30093fe4b.jpg'
abbrlink: 2ef385a5
---
### 1.1 xpath语法

```python
xpath基本语法：
	1.路径查询
    // :查找所有子孙节点，不考虑层级关系
    /  :直接找子节点
    2.谓词查询
    //div[@id]
    //div[@id="hdj"]
    3.属性查询
    //@class
    4.模糊查询
    //div[contains(@id,"he")]
    5.内容查询
    //div/h1/text()
    
```

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
    <ul>
        <li id="l1" class="c1">四川</li>
        <li id="l2">重庆</li>
        <li>贵州</li>
    </ul>
  <ul>
        <li>成都</li>
        <li>湖南</li>
        <li>白净</li>
    </ul>

</body>
</html>

查找ul下面的li 

xpath   ://body/ul/li

查找 li下面所有有id属性的li标签,text()获取标签中内容
xpath   ://ul/li[@id]/text()

找到标签为li的标签
xpath   ://ul/li[@id="l1"]

查找到id 为l1的li标签的class的属性值
//ul/li[@id="l1"]/@clas
```



