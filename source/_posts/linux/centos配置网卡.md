---
title: centos配置静态IP
cover: 'https://s2.loli.net/2023/01/26/8LdsQPwqIVamYfH.jpg'
tags: 静态IP
categories: Linux
top_img: 'https://s2.loli.net/2023/01/26/8LdsQPwqIVamYfH.jpg'
abbrlink: 7f72d94
---


## centos配置静态IP

- 使用进入vim 进入 /etc/sysconfig/network-scripts/ifcfg-ens3

```shell
TYPE="Ethernet"
PROXY_METHOD="none"
BROWSER_ONLY="no"
BOOTPROTO="none"
DEFROUTE="yes"
IPV4_FAILURE_FATAL="yes"
IPV6INIT="yes"
IPV6_AUTOCONF="yes"
IPV6_DEFROUTE="yes"
IPV6_FAILURE_FATAL="no"
IPV6_ADDR_GEN_MODE="stable-privacy"
NAME="ens33"
UUID="44f6e5e3-cb8d-4148-a480-d2e75bc6a8a4"
DEVICE="ens33"
ONBOOT="yes"
IPADDR="192.168.169.110"
NETMASk="255.255.255.0"
GATEWAY="192.168.169.2"
DNS1="8.8.8.8"
IPV6_PRIVACY="no"

```

```shell
#重启网卡
systemctl restart network
```

![image-20230127100554113](https://s2.loli.net/2023/01/27/TUPKYj3ehoq8fAR.png)

2.输入ip addr查看是否获取到IP地址，ping www.baidu.com。

