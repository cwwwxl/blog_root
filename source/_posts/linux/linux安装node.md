---
title: linux安装node
cover: 'https://s2.loli.net/2023/04/28/mMFQupEfqWcd5Oe.png'
tags: linux
categories: node
top_img: 'https://s2.loli.net/2023/04/28/mMFQupEfqWcd5Oe.png'
abbrlink: 3dad8e04
---
# linux安装node

#### 1、下载安装包

```shell
wget http://nodejs.org/dist/v0.10.25/node-v0.10.25.tar.gz
版本根据自己需要进行选择。
```



#### 2、安装gcc

```shell
yum install gcc openssl-devel gcc-c++ compat-gcc-34 compat-gcc-34-c++
```



#### 3、解压node

```shell
tar -xf node-v0.10.25.tar.gz
3、进入node目录
cd node-v0.10.25
```



#### 4、配置目录

```shell
./configure --prefix=/usr/local/node
```



#### 5、安装npm

```shell
make && make install
```



#### 6、加入链接

```shell
ln -s /usr/local/node/bin/* /usr/sbin/
```



#### 7、测试时候安装成功

```shell
node -v
npm -v


```

