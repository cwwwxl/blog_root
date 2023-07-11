---
title: httpd解决方案
cover: 'https://s2.loli.net/2023/01/26/uPStV1NECwacO6s.jpg'
tags: openstack
categories: 云计算
top_img: 'https://s2.loli.net/2023/01/26/jFldL1tuzWbvpyq.jpg'
abbrlink: d8a1ddad
sticky:
---
## 1.httpd解决方案

​	Job for httpd.service failed because the control process exited with error code.see “systemctl status httpd.service” and “journalctl -xe” for details.

这个提醒 会让你去查看当前状态日志，查看状态日志时基本上你会发现 一些服务处在 fail 状态 基本上就是httpd服务没能够起来。

```shell
1.杀掉这个进程就可以：

先ps –aux | grep http
第二行数字就是进程pid号
Kill -9 pid号 即可杀掉进程
杀完之后 重启reboot 再ps –aux | grep http 确保httpd进程被杀掉 若发现无法杀掉进程 往下看

2. 如果杀掉进程的话，重启进程还存在，就代表成了僵尸进程，就重新安装哈httpd
步骤如下：
先用yum卸载httpd和mod_wsgi，
命令：yum remove httpd mod_wsgi;
然后再安装yun -y install httpd mod_wsgi 就可以了
安装完成后 service httpd restart 即可 问题基本排除

```

```shell
#查看运行状态
systemctl status httpd.service
#启动服务
systemctl start httpd.service
```

### 2.Openstack glance 安装 403错误

​	执行openstack endpoint create --region RegionOne image public http://controller160:9292

报错HTTP 403 Forbidden:You are not authorized to complete publicize_image action

**解决办法**

- 修改glance配置文件 

```shell
vim /etc/glance/glance-api.conf
 
vim /etc/glance/glance-registry.conf
```

- 出现这种错误一般是配置文件中flavor=keystone没有写，2个文件都要看

```shell
[paste_deploy]
flavor = keystone
```

- 重新启动服务

```shell
service openstack-glance-api restart
service openstack-glance-registry restart
```

### openstack错误

```shell
   platform: Linux-3.10.0-1127.el7.x86_64-x86_64-with-centos-7.8.2003-Core
        pip: 9.0.3
        setuptools: 33.1.1
        setuptools_rust: 1.1.2
        rustc: n/a
        =============================DEBUG ASSISTANCE=============================

    Traceback (most recent call last):
      File "<string>", line 1, in <module>
      File "/tmp/pip-build-9859g78y/cryptography/setup.py", line 63, in <module>
        rust_version=">=1.48.0",
      File "/usr/lib64/python3.6/distutils/core.py", line 108, in setup
        _setup_distribution = dist = klass(attrs)
      File "/usr/local/lib/python3.6/site-packages/setuptools/dist.py", line 320, in __init__
        _Distribution.__init__(self, attrs)
      File "/usr/lib64/python3.6/distutils/dist.py", line 261, in __init__
        warnings.warn(msg)
    UserWarning: Unknown distribution option: 'cffi_modules'

    ----------------------------------------
Command "python setup.py egg_info" failed with error code 1 in /tmp/pip-build-9859g78y/cryptography/

```

### 解决方案



```shell
 pip3 install setuptools_rust
 sudo pip3 install setuptools==33.1.1


yum install gcc libffi-devel python-devel openssl-devel -y

python3 -m pip install --upgrade pip

pip3 install paramiko

```

