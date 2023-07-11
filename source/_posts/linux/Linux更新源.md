---
title: Linux更新源
cover: 'https://s2.loli.net/2023/01/22/YZBW2uxLr9hlFK3.jpg'
tags: ubuntu
categories: Linux
top_img: 'http://rnwft8xfc.hn-bkt.clouddn.com/%E5%9B%BE%E7%89%87/151472.jpg'
abbrlink: b7866dc2
---
#### Linux更新源

```shell
sudo apt-get install vim 
```

![image-20230107163536689](https://s2.loli.net/2023/01/07/lzXfgEcaWh1w6pZ.png)

```shell
sudo vim /etc/apt/sources.list
```

```shell
deb http://mirrors.aliyun.com/ubuntu/ bionic main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ bionic-security main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ bionic-updates main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ bionic-proposed main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ bionic-backports main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ bionic main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ bionic-security main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ bionic-updates main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ bionic-proposed main restricted universe multiverse
deb-src 
```

```shell
wq保存退出
更新 sudo apt-get update
```

