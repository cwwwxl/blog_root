---
title: select查询语句
cover: 'https://s2.loli.net/2023/01/04/xG2nKtYRwc1mUWk.jpg'
tags: mysql
categories: 数据库
top_img: 'http://rnwft8xfc.hn-bkt.clouddn.com/%E5%9B%BE%E7%89%87/151472.jpg'
abbrlink: 24fbc361
---
# 3-1 导入/导出sql文件

```mysql
-- 将数据库中的数据导出，在命令行直接写命令，不需要进入MySQL
mysqldump -u root -p --all-databases > 本地路径  -- 将数据库中所有的库和库中数据全部导出到本地
C:\Users\12801>mysqldump -u root -p --all-databases > F:\all_data.sql

mysqldump -u root -p 库名 > 本地路径  -- 将数据库中指定的库的数据导出到本地
C:\Users\12801>mysqldump -u root -p test > F:\test.sql

mysqldump -u 用户名 -p 数据库名 表名1 表名2 ...> 本地路径  -- 将库中的一个表或者多个表和表中的数据导出到本地
C:\Users\12801>mysqldump -u root -p1026 test t1 t2 > F:\many_tables.sql

-- 向数据库中导入sql文件，需要登录mysql
source sql文件  -- 导入所有库
-- 导入一个库文件或者表需要先进入到库里
    use 库;
    source sql文件
```

# 3-2 select介绍

```mysql
-- DQL 数据查询语言，并不是官方定义的，只是因为数据查询非常重要而且知识点较多，所以就出现了DQL这个分类
-- 功能
	获取表中的数据行，可以单独使用也可以配合其他子句使用，标准用法是配合其他子句使用
-- 单独使用
	select localtime();  -- 配合内置函数使用，显示当前时间
	select concat(id, '@', name) from t1;  -- concat拼接操作
	
-- 标准用法：配合from /where /group by /having /order by /limit子句进行查询数据
-- 上述子句的顺序就是SQL语法的书写顺序，需要注意语法书写顺序和执行顺序不一致
```

# 3-3 单表查询 - from

```mysql
-- 表准备
在我要自学网本课程的资源下载中下载课程中用到的sql文件
-- 将数据导入到本地MySQL中
source sql文件路径

-- select配合from子句：select 想要查询的数据 from 表;
select * from dql.info;   -- 查询info表中的所有数据，全表扫描
select id, name from dql.info;  -- 查询city表中id和name的数据，查询部分列的值
ps：通常不使用全表扫描的方式查询数据，在数据非常多的情况下全表扫描是非常慢的
```

# 3-4 单表查询 - where

```mysql
-- 作用：where字句后需要有筛选条件，根据条件筛选出满足的数据
-- 配合= > < <= >= != 比较运算使用
	select * from info where teacher='李老师';  -- 查询李老师带的学生的所有信息

-- 配合and or not 逻辑判断使用
	select * from info where score>90 and age<8; -- 查询成绩大于90分并且年龄小于8岁的学生信息
	select * from info where score>90 or score<60 -- 查询成绩大于90分或者小于60分的学生的信息
	
-- 配合between and 范围使用
	select * from info where score between 80 and 100; -- 查询分数在80-100范围的学生信息
	
-- 配合in、not in 成员运算使用
 	select * from info where teacher in('张老师', '李老师'); -- 查询张老师和李老师带的学生信息
 	select * from info where teacher not in ('李老师', '刘老师'); -- 查询没有被李老师和刘老师带的学生的信息
 	
-- 配合like 模糊查询使用
	-- % 匹配任意多个字符
	-- _匹配任意单个字符
	SELECT * FROM info WHERE name LIKE '小%';  -- 查询'小'开头的学生信息
	SELECT * FROM city WHERE district LIKE '小_';  -- 查询'小'开头的学生信息
```

# 3-5 单表查询 - group by

```mysql
-- 作用：按照某种逻辑分类分组，然后以组为单位统计分析数据。根据情况判断是否需要where过滤
-- 使用：分组后不能使用表内定义字段作为select后的查询字段，需要配合聚合函数使用；由于MySQL数据库中不允许一行数据与多行数据对应，因此需要使用聚合函数将多个数据整合为与分组依据对应的一个数据。
-- max()/min()最大/最小
	select teacher, min(score) from info group by teacher; -- 每个老师所带的学生的最低分数
-- avg() 平均数
	select grade, avg(score) from info group by grade; -- 每个年级的平均分
-- count() 计数
	select teacher, count(id) from info group by teacher; -- 统计每个老师带的学生数量
-- sum() 总和
	select grade, sum(score) from info group by grade; -- 统计每个年级学生的总分
-- group_concat()，列转行，将group by产生的同一个分组中的值连接起来，返回一个字符串结果。group_concat函数首先根据group by指定的列进行分组，将同一组的列转换成行出来。
	select grade, avg(score)， group_concat(name) from info group by grade;-- 统计每个年级的平均分，并且列出每个年级学生的名字
	
-- 补充：可以使用as给字段起别名，as可以省略
	select grade as '年级', avg(score) as '平均分' from info group by grade; -- 每个年级的平均分
	
-- 和where子句结合使用
	select grade, age, avg(score) as '平均分' from info where grade=1 group by age; -- 统计一年级中不同年龄的分数平均值
	
-- 使用分组group by注意事项
<1> 关键字where和group by同时出现的时候group by必须在where的后面
<2> where先对整体数据进行过滤之后再分组操作
<3> where后面的筛选条件不能使用聚合函数
```

# 3-6 单表查询 - having

```mysql
-- 作用：对分组之后的数据再次进行筛选，作用和where类似，但是在having之后可以直接使用聚合函数作为条件
-- 使用
	select grade, avg(score) from info group by grade having avg(score)>=80; -- 统计每个年级的平均分并且输出平均分大于等于80的年级
	
-- from + where + group by + having结合使用
	select grade, age, avg(score) as '平均分' from info where grade=1 group by age having avg(score)>85;-- 统计一年级中不同年龄学生的分数平均值并输出平均分大于85分的年龄和分数
```

# 3-7 单表查询 - distinct

```mysql
-- 作用：对跟在distinct之后的字段数据去重操作，数据必须完全一致才能去重，如果去重的数据包含主键是无法进行去重的操作的
-- 使用：
	select distinct id, teacher from info;  -- 查询老师和id,无法去重
	select distinct teacher from info;  -- 只查询老师可以去重
	select distinct age from info; -- 查询学生年龄都有哪些
-- 结合group by使用
	select grade, min(score) as '最低分数', group_concat(distinct teacher) from info group by grade;-- 查询每个年级最低分数并且输出该年级所有老师
```

# 3-8 单表查询 - order by

```mysql
-- 作用：按照指定顺序输出结果，默认是从小到大，desc是从大到小
-- 使用：
	select * from info order by id desc; -- 将数据按照id从大到小的顺序排序
	select * from info order by name; -- 按照学生姓名进行排序，非数字的排序是按照ASCII码表的顺序进行排序
	select name, score from info where grade=1 order by score desc;-- 查询1年级的学生的姓名和成绩，并按照学生成绩从大到小进行排序
```

# 3-9 单表查询 - limit

```mysql
-- 作用：限制数据的展示条数，通常配合order by使用
-- 使用
	-- limit后跟一个数字，表示从头开始展示的条数
	select * from info limit 5;  -- 展示前5条数据
	-- limit后跟两个数字，第一个参数是起始位置，第二个参数是展示的条数
	select * from info limit 5, 6; --  从第6条开始显示，共显示6条LIMIT N, M;  跳过N行，显示M行
	select * from info limit 6 offset 5; --  从第6条开始显示，共显示6条，LIMIT M OFFSET N; 跳过N行，显示M行
	
	-- 配合order by使用
	select * from info order by id desc limit 5; -- 将数据按照id从大到小的顺序排序，并展示前5条数据
	select name, score from info where grade=1 order by score desc limit 2, 4;-- 查询1年级的学生的姓名和成绩，并按照学生成绩从大到小进行排序，从第2条开始展示，展示4条
		select name, score from info where grade=1 order by score desc limit 4 offset 2;-- 查询1年级的学生的姓名和成绩，并按照学生成绩从大到小进行排序，从第2条开始展示，展示4条
```

# 3-10 单表查询总结

```mysql
-- 单表查询所有子句书写顺序
SELECT DISTINCT field_list
FROM table
WHERE <where_condition>
GROUP BY <group_by_list>
HAVING <having_condition>
ORDER BY <order_by_condition>
LIMIT <limit_number>

-- 执行顺序
from		# 找到表
where		# 筛选
group by	# 分组
having		# 再筛选
select		# 选择查看内容
distinct	# 内容去重
order by	# 内容排序
limit		# 限制内容条数

-- 单表查询总结
 where + 筛选条件; 可以配合= > < <= >= != 比较运算使用，也可以配合配合and or not 逻辑判断使用，也可以配合between and 范围使用，还有配合in、not in 成员运算使用以及配合like 模糊查询使用(% 匹配任意多个字符,_匹配任意单个字符)
	
 group by 通常都是和聚合函数min()/max()/avg()/count()/sum()一起使用，分组后不能使用表内定义字段作为select后的查询字段，需要配合聚合函数使用；由于MySQL数据库中不允许一行数据与多行数据对应，因此需要使用聚合函数将多个数据整合为与分组依据对应的一个数据。
 
 having 对分组之后的数据再次进行筛选，作用和where类似，但是在having之后可以直接使用聚合函数作为条件,只能在分组之后使用，如果没有使用分组筛选条件只能使用where
 
 distinct 对跟在distinct之后的字段数据去重操作，数据必须完全一致才能去重，如果去重的数据包含主键是无法进行去重的操作的
 
 order by 按照指定顺序输出结果，默认是从小到大，desc是从大到小
 
 limit 限制数据的展示条数，通常配合order by使用，有三种写法
 	limit N; -- 展示结果的前N条数据
 	limit N, M; -- 跳过N条数据之后再展示M条数据
 	LIMIT M OFFSET N; 跳过N行，显示M行
```

# 3-11 单表查询综合实战(1)

```mysql
数据准备：https://downloads.mysql.com/docs/world-db.zip
或者在我要自学网上进行下载
-- 查询中国所有城市信息
select * from city where countrycode='CHN';
-- 中国人口大于5000000的城市信息
SELECT * FROM city WHERE countrycode='CHN' AND population>5000000;
-- 查询中国省的名字前面带guang开头的
SELECT * FROM city WHERE district LIKE 'guang%';
-- 查询中国和美国所有城市信息
SELECT * FROM city WHERE countrycode IN ('CHN' ,'USA'); 
-- 查询世界上人口数量大于100w小于200w的城市信息
SELECT * FROM city WHERE population >1000000 AND population <2000000;  -- 第一种
SELECT * FROM city WHERE population BETWEEN 1000000 AND 2000000;  -- 第二种
-- 统计世界上每个国家的总人口数.
SELECT countrycode ,SUM(population) FROM  city  GROUP BY countrycode;
-- 统计中国每个省的人口数
SELECT district,SUM(Population) FROM city  WHERE countrycode='chn' GROUP BY district;
```

# 3-12 单表查询综合实战(2)

```mysql
-- 统计中国每个省的名字，人数，城市个数，城市名字
SELECT
	district AS '省份',
	sum( population ) AS '总人数',
	count( id ) AS '城市个数',
	GROUP_CONCAT( NAME ) as '城市名称' 
FROM
	city 
WHERE
	CountryCode = 'CHN' 
GROUP BY
	District;
-- 统计中国每个省的总人口数，只打印总人口数小于1000000
SELECT
	District AS '省份',
	SUM( Population ) AS '总人口数'
FROM
	city 
WHERE
	CountryCode = 'CHN' 
GROUP BY
	District 
HAVING
	SUM( Population ) < 1000000;
	
-- 统计中国每个省的总人口数，只打印总人口数大于1000000,从大到小排列，只显示前6~11条。
SELECT
	District AS '省份',
	SUM( Population ) AS '总人口数' 
FROM
	city 
WHERE
	CountryCode = 'CHN' 
GROUP BY
	District 
HAVING
	SUM( Population )> 1000000 
ORDER BY
	SUM( Population ) DESC 
	LIMIT 5,
	6;

```



