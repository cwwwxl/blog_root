---
title: mysql远程登录
cover: 'https://s2.loli.net/2023/01/22/aGcyWoTErqIj1ts.jpg'
tags: mysql
categories: 数据库
top_img: 'http://rnwft8xfc.hn-bkt.clouddn.com/%E5%9B%BE%E7%89%87/151472.jpg'
abbrlink: c93102b7
---
### 1. mysql开启远程登录

```sql
#删除之前配置
drop user 'root'@'%';
#配置远程登录
CREATE USER 'root'@'%' IDENTIFIED BY '123456'; 
GRANT ALL ON *.* TO 'root'@'%'; 
ALTER USER 'root'@'%' IDENTIFIED WITH mysql_native_password BY '12345';
```

### 2.刷新权限

```sql
FLUSH PRIVILEGES;
sudo yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
sudo yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
```

