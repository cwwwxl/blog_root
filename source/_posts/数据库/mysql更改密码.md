---
title: mysql更改密码
cover: 'https://s2.loli.net/2023/01/22/83ovLZMwTCuSUO6.jpg'
tags: mysql
categories: 数据库
top_img: 'http://rnwft8xfc.hn-bkt.clouddn.com/%E5%9B%BE%E7%89%87/151472.jpg'
abbrlink: 37f0288c
---
### mysql更改密码

进入到MySQL输入以下指令

<img src="https://s2.loli.net/2023/01/07/WyHISEZt9JnGlQz.png" alt="image-20230107162204484" style="zoom:80%;" />

```mysql
mysql> use mysql;
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A
Database changed

mysql> update mysql.user set authentication_string=password('123456') where user='root' and Host ='localhost';
Query OK, 1 row affected, 1 warning (0.04 sec)
Rows matched: 1  Changed: 1  Warnings: 1

mysql> update user set plugin="mysql_native_password"; 
Query OK, 0 rows affected (0.00 sec)
Rows matched: 4  Changed: 0  Warnings: 0

mysql> flush privileges;
Query OK, 0 rows affected (0.12 sec)

ALTER USER “root”@“localhost” IDENTIFIED WITH mysql_native_password BY “your password”;
```

```mysql
show databases;  #查看数据库
use mysql;       #使用数据库

        

update user set authentication_string=PASSWORD("123456") where user='root';  

#修改密码，密码设置的是 123456



update user set plugin="mysql_native_password";

flush privileges; #刷新

exit;  #退出,密码已经修改成功

/etc/init.d/mysql restart   #重启mysql
```

```mysql
重置为新的密码，这里分为两个版本

一直使用5.7的命令在8.0的版本中执行，一直报错，感到很诧异，后查找资料后发现，mysql 5.7.9以后废弃了password字段和password()函数；

而且用于表示用户密码的authentication_string字段只能是mysql加密后的41位字符串密码。因此在修改密码时，5.7版本和8.0版本有所不同，需要使用不同的语句执行，其他版本也有可能不同。

1、MySql5.7

update user set authentication_string = password(["your new password"]) where user = "your username" [and Host="localhost"];

2、MySql8.0

（1）检查authentication_string字段是否为空，不为空先置空，我比较懒，我就直接置空了。

use mysql;
update user set authentication_string='' where user='root';
ALTER user 'root'@'localhost' IDENTIFIED BY '123456';

（2）如果顺利执行，那么恭喜你已经修改密码成功了，如果报错则执行flush privileges;刷新MySQL的系统权限相关表后再次执行上一步。
```

