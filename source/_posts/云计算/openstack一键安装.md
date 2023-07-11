---
title: openstack一键安装脚本
#sticky: 1
cover: 'https://s2.loli.net/2023/03/24/Oh5kPXLzcuybCRU.png'
tags: openstack
categories: 云计算
top_img: 'https://s2.loli.net/2023/03/24/Oh5kPXLzcuybCRU.png'
abbrlink: a1b82ddb
---
## 使用Packstack工具一键安装OpenStack

## 1.环境规划

| **操作系统版本** | **硬件配置**       | **IP地址规划** | **主机名** | **虚拟机软件**       | **OpenStack版本** |
| ---------------- | ------------------ | -------------- | ---------- | -------------------- | ----------------- |
| centos7.9        | 4vCPUS/15G 30G硬盘 | 172.17.2.60/24 | openstack  | VMware WorkStation16 | Stein             |

centos7.9镜像获取地址：http://mirrors.aliyun.com/centos/7.9.2009/isos/x86_64/CentOS-7-x86_64-Minimal-2009.iso

## 2.准备资料

OpenStack官方文档：https://docs.openstack.org/install-guide/

使用Packstack一键部署OpenStack的方式，隐藏了技术细节，如果使用中出来任何问题，我们都无法解决，所以体验完OpenStack的日常界面操作后，后期还是要仔细查看官方文档，然后一步步手工安装OpenStack各组件。

阿里云yum源设置官方文档：https://developer.aliyun.com/mirror/centos?spm=a2c6h.13651102.0.0.3e221b11Jpkdzb

### 3.修改centos系统的主机名

```shell
hostnamectl set-hostname openstack
```

### 4.设置centos系统为静态IP地址

```shell
#注意修改自己的网卡配置文件
vi /etc/sysconfig/network-scripts/ifcfg-ens32
#重启网络服务
systemctl restart network
#验证IP地址
ip addr
```

## 5.添加主机hosts记录

```shell
echo "172.17.2.60 openstack" >> /etc/hosts
#验证修改结果
more /etc/hosts
```

### 6.设置本机SSH免密码登录

```shell
#生成ssh密钥
ssh-keygen
#添加密钥信息到~./ssh/know_hosts文件
ssh-copy-id root@172.17.2.60
```

### 7.关闭防火墙

```shell
systemctl stop firewalld
systemctl disable firewalld
#查看防火墙状态
systemctl status firewalld
```

### 8.关闭SeLinux

```shell
setenforce 0
sed -i 's/SELINUX=enforcing/SELINUX=disabled/' /etc/selinux/config
```

### 9.关闭NetworkManager服务

```shell
systemctl stop NetworkManager
systemctl disable NetworkManager
#查看NetworkManager状态
systemctl status NetworkManager
```

### 10.修改官方yum源为阿里云yum源

```shell
curl -o /etc/yum.repos.d/CentOS-Base.repo https://mirrors.aliyun.com/repo/Centos-7.repo
#重建本地yum索引缓存
yum makecache
```

### 11.系统升级

```shell
#升级
yum -y update
#重启
reboot
```

## 12.安装常用的软件

```shell
 yum -y install vim bash-completion yum-utils
```

### 13.安装`OpenStack Stein`的`yum`库

```shell
yum -y install centos-release-openstack-stein
```

### 14.修改`CentOS-OpenStack-stein.repo`配置文件

```shell
cd /etc/yum.repos.d/
#备份
cp -a CentOS-OpenStack-stein.repo CentOS-OpenStack-stein.repo.bak

#修改配置文件
vim CentOS-OpenStack-stein.repo
[centos-openstack-stein]
baseurl=http://mirrors.aliyun.com/$contentdir/$releasever/cloud/$basearch/openstack-stein/
#mirrorlist=http://mirrorlist.centos.org/?release=$releasever&arch=$basearch&repo=cloud-openstack-stein
...

#清除本地yum索引缓存，然后再重建索引缓存
yum clean all && yum makecache
```

### 15.安装packstack工具

```shell
yum -y install openstack-packstack
```

### 16.安装OpenStack allinone

```shell
packstack --allinone
```

自动化安装时间很长，静静等待，直到出现如下信息，表示成功安装。

```shell
**** Installation completed successfully ******

Additional information:
 * Parameter CONFIG_NEUTRON_L2_AGENT: You have choosen OVN neutron backend. Note that this backend does not support LBaaS, VPNaaS or FWaaS services. Geneve will be used as encapsulation method for tenant networks
 * A new answerfile was created in: /root/packstack-answers-20220222-171022.txt
 * Time synchronization installation was skipped. Please note that unsynchronized time on server instances might be problem for some OpenStack components.
 * File /root/keystonerc_admin has been created on OpenStack client host 172.17.2.60. To use the command line tools you need to source the file.
 * To access the OpenStack Dashboard browse to http://172.17.2.60/dashboard .
Please, find your login credentials stored in the keystonerc_admin in your home directory.
 * The installation log file is available at: /var/tmp/packstack/20220222-171021-YFXtB3/openstack-setup.log
 * The generated manifests are available at: /var/tmp/packstack/20220222-171021-YFXtB3/manifests
```

### 17 .查看Dashboard web页面的登录账号及密码

```shell
cat keystonerc_admin 
unset OS_SERVICE_TOKEN
    export OS_USERNAME=admin		#Dashboard登录账号
    export OS_PASSWORD='12b2ca7e963242ae'	#Dashboard登录密码
    export OS_REGION_NAME=RegionOne
    export OS_AUTH_URL=http://172.17.2.60:5000/v3
    export PS1='[\u@\h \W(keystone_admin)]\$ '
    
export OS_PROJECT_NAME=admin
export OS_USER_DOMAIN_NAME=Default
export OS_PROJECT_DOMAIN_NAME=Default
export OS_IDENTITY_API_VERSION=3
```



### 18.修改密码

```shell
vim keystonerc_admin
export OS_PASSWORD='admin'	#填入修改后的新密码
```

### 19.本人亲测没有问题

![image-20230129181318202](https://s2.loli.net/2023/01/29/ys49ZU6JiKDtSvl.png)