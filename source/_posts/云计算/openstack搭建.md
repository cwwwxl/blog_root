---
title: openstack搭建

cover: 'https://s2.loli.net/2023/02/22/54KCUJ6sgS1pr3V.jpg'
tags: openstack
categories: 云计算
top_img: 'https://s2.loli.net/2023/02/22/54KCUJ6sgS1pr3V.jpg'
abbrlink: d0acfaca
---
## 一.OpenEuler镜像

### 1.1 镜像

- 下载j最新版的OpenEuler，在 https://repo.huaweicloud.com/openeuler/ 下载 [openEuler-22.03-LTS-x86_64-dvd.iso](https://repo.huaweicloud.com/openeuler/openEuler-22.03-LTS/ISO/x86_64/openEuler-22.03-LTS-x86_64-dvd.iso)

- 采用默认安装(最小化安装)即可，其中超级用户root的密码设置为 openeuler!23456 (系统要求用户密码长度至少8位，至少有3种字符)。

| 控制节点 controller | IP:10.0.0.10 | 认证服务镜像服务监控服务计算服务网络服务Dashboard服务 | httpd, mariadb, memcached, rabbitmq-server, etcd,  openstack-glance-api, openstack-nova-api, openstack-nova-scheduler, openstack-nova-conductor, openstack-nova-novncproxy,neutron-server,neutron-linuxbridge-agent,neutron-dhcp-agent,neutron-metadata-agent,neutron-l3-agent |
| ------------------- | ------------ | ----------------------------------------------------- | ------------------------------------------------------------ |
| 计算节点 compute-1  | IP:10.0.0.11 | 计算服务网络服务                                      | openstack-nova-compute,neutron-linuxbridge-agent             |

### 1.2控制节点

- 设置网卡

![image-20230120154904058](https://s2.loli.net/2023/01/20/xGfeIinHTvBJjs9.png)

![image-20230120154918889](https://s2.loli.net/2023/01/20/7yEXHURf6qsiO2l.png)

![image-20230120154934614](https://s2.loli.net/2023/01/20/5SmXFCOJyz9kLfs.png)

```shell
hostnamectl set-hostname controller
echo "10.0.0.10 controller" >> /etc/hosts
echo "10.0.0.11 compute-1" >> /etc/hosts

sed -i "s/BOOTPROTO=dhcp/BOOTPROTO=static/g" /etc/sysconfig/network-scripts/ifcfg-ens33
sed -i "s/ONBOOT=no/ONBOOT=yes/g" /etc/sysconfig/network-scripts/ifcfg-ens33
echo "IPADDR=10.0.0.10" >> /etc/sysconfig/network-scripts/ifcfg-ens33
echo "NETMASK=255.255.255.0" >> /etc/sysconfig/network-scripts/ifcfg-ens33
#echo "PREFIX=24" >> /etc/sysconfig/netnwork-scripts/ifcfg-ens33
echo "GATEWAY=10.0.0.2" >> /etc/sysconfig/network-scripts/ifcfg-ens33
echo "DNS1=223.5.5.5" >> /etc/sysconfig/network-scripts/ifcfg-ens33
echo "DNS2=8.8.8.8" >> /etc/sysconfig/network-scripts/ifcfg-ens33
nmcli c reload
nmcli c up ens33
```

controller 为控制节点的域名，compute-1 为计算节点的域名

10.0.0.10为本地的IP址，255.255.255.0 为子网掩码，10.0.0.2 为网关地址， 223.5.5.5 / 223.6.6.6 这是阿里云提供的DNS服务器，8.8.8.8 这是google提供的DNS服务器。nmcli 为网络服务的控制命令(network 服务不是默认，已经弃用，可以安装)

设置DNS的方法除了在网卡设置上DNS1=x.x.x.x ，还可以编辑文件“/etc/resolv.conf” ，验证DNS 

```shell
echo "nameserver 223.5.5.5" >> /etc/resolv.conf
echo "nameserver 8.8.8.8" >> /etc/resolv.conf
```

- 验证dns

```shell
dig alidns.com
```

### 1.3 关闭swap分区，关闭SeLinux

```shell
swapoff -a
sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab
setenforce 0
sed -i 's#SELINUX=enforcing#SELINUX=disabled#g' /etc/selinux/config
systemctl disable firewalld && systemctl stop firewalld

```

### 1.4 时间服务

```shell
sed -i '/^pool/d' /etc/chrony.conf
echo "pool ntp.aliyun.com iburst" >> /etc/chrony.conf
timedatectl set-timezone Asia/Shanghai
systemctl restart chronyd
chronyc sources
```

- ntp.aliyun.com 为阿里的NTP服务器，用于网络同步时间

### 1.5 修改内核参数

```shell
echo "net.ipv4.ip_nonlocal_bind=1" >> /etc/sysctl.conf
echo "net.ipv4.ip_forward=1" >> /etc/sysctl.conf
sysctl -p 
reboot
```

### 1.6 软件源（CentOS-stream）

```shell
yum install centos-release-openstack-train -y 安装t版本openstack
relese 

#yum install python-openstackclient -y
#yum install openstack-selinux -y
```

### 1.7 基础组件

```shell
yum install -y openstack-release-wallaby 
#yum install openstack-selinux -y
yum install -y openstack-release-wallaby
#ln -s /usr/bin/python3 /usr/bin/python
git clone https://github.com/pixelb/crudini.git
cd crudini
python3 setup.py install
#ln -s ~/crudini /usr/bin/crudini

#yum config-manager --set-enabled PowerTools
```

crudini的工具，在OpenEuler的源上是没有这个工具的，可以下载其源代码进行安装使用，本地应用使用python38进行安装使用。若是在CentOS8的数据源，可以直接安装，不需要下载源代码进行安装

```shell
yum install -y crudini
```

### 1.8 数据库Mariadb

```shell
yum install -y mariadb mariadb-server python3-PyMySQL
tee /etc/my.cnf.d/openstack.cnf <<-'EOF'
[mysqld]
default-storage-engine = innodb
innodb_file_per_table = on
max_connections = 4096
collation-server = utf8_general_ci
character-set-server = utf8
EOF
systemctl enable mariadb && systemctl start mariadb
echo -e "Y\n123456\n123456\nY\nn\nY\nY\n" | mysql_secure_installation
```

设置mariadb的超级用户root的密码为123456

### 1.9 队列服务RabbitMQ

```shell
yum install -y rabbitmq-server
systemctl enable rabbitmq-server && systemctl start rabbitmq-server
rabbitmqctl add_user openstack 123456
rabbitmqctl set_permissions openstack ".*" ".*" ".*"
rabbitmq-plugins list

rabbitmq-plugins enable rabbitmq_management rabbitmq_management_agent
ss -tnl
```

​	开启图形化管理(rabbitmq_management,rabbitmq_management_agent)

ss -tnl 查看当前端口的使用情况。

### 2.0 缓存服务Memcached

```shell
yum install -y memcached python3-memcached
sed -i "s/-l 127.0.0.1,::1/-l 127.0.0.1,::1,controller/g" /etc/sysconfig/memcached
systemctl enable memcached && systemctl start memcached

```

### 2.1 键值对存储服务Etcd

```shell
yum install -y etcd
sed -i.default -e '/^#/d' -e '/^$/d' /etc/etcd/etcd.conf
rm -f /etc/etcd/etcd.conf
tee /etc/etcd/etcd.conf <<-'EOF'
#[Member]
ETCD_NAME="controller"
ETCD_DATA_DIR="/var/lib/etcd/default.etcd"
ETCD_LISTEN_PEER_URLS="http://0.0.0.0:2380"
#ETCD_LISTEN_PEER_URLS="http://10.0.0.10:2380"
ETCD_LISTEN_CLIENT_URLS="http://192.168.169.200:2379"

#[Clustering]
ETCD_INITIAL_ADVERTISE_PEER_URLS="http://192.168.169.200:2380"
ETCD_ADVERTISE_CLIENT_URLS="http://192.168.169.200:2379"
ETCD_INITIAL_CLUSTER="controller=http://192.168.169.200:2380"
ETCD_INITIAL_CLUSTER_TOKEN="etcd-cluster-01"
ETCD_INITIAL_CLUSTER_STATE="new"
EOF
systemctl enable etcd 
systemctl start etcd
```

### 2.1认证服务keystone(Identity Service)

```shell
mysql -uroot -p123456 -e "CREATE DATABASE keystone"
mysql -uroot -p123456 -e "GRANT ALL PRIVILEGES ON keystone.* TO 'keystone'@'localhost' IDENTIFIED BY '123456'"
mysql -uroot -p123456 -e "GRANT ALL PRIVILEGES ON keystone.* TO 'keystone'@'%' IDENTIFIED BY '123456'"
yum install -y openstack-keystone httpd python3-mod_wsgi

sed -i.default -e '/^#/d' -e '/^$/d' /etc/keystone/keystone.conf
crudini --set /etc/keystone/keystone.conf database connection mysql+pymysql://keystone:123456@controller/keystone
crudini --set /etc/keystone/keystone.conf token provider fernet
su -s /bin/sh -c "keystone-manage db_sync" keystone
keystone-manage fernet_setup --keystone-user keystone --keystone-group keystone
keystone-manage credential_setup --keystone-user keystone --keystone-group keystone
keystone-manage bootstrap --bootstrap-password 123456 \
 --bootstrap-admin-url http://controller:5000/v3/ \
 --bootstrap-internal-url http://controller:5000/v3/ \
 --bootstrap-public-url http://controller:5000/v3/ \
 --bootstrap-region-id RegionOne

echo "ServerName controller" >> /etc/httpd/conf/httpd.conf
ln -s /usr/share/keystone/wsgi-keystone.conf /etc/httpd/conf.d/
systemctl enable httpd && systemctl start httpd
echo "export OS_USERNAME=admin" >> /etc/profile
echo "export OS_PASSWORD=123456" >> /etc/profile
echo "export OS_PROJECT_NAME=admin" >> /etc/profile
echo "export OS_USER_DOMAIN_NAME=Default" >> /etc/profile
echo "export OS_PROJECT_DOMAIN_NAME=Default" >> /etc/profile
echo "export OS_AUTH_URL=http://controller:5000/v3" >> /etc/profile
echo "export OS_IDENTITY_API_VERSION=3" >> /etc/profile
source /etc/profile
yum install -y python2-openstackclient
openstack project create --domain default --description "Service Project" service
```

### 2.2核验身份

```shell
openstack token issue

systemctl status httpd memcached etcd rabbitmq-server mariadb
```

### 2.3 镜像服务glance(Image Service)

```shell
mysql -uroot -p123456 -e "CREATE DATABASE glance"
mysql -uroot -p123456 -e "GRANT ALL PRIVILEGES ON glance.* TO 'glance'@'localhost' IDENTIFIED BY '123456'"
mysql -uroot -p123456 -e "GRANT ALL PRIVILEGES ON glance.* TO 'glance'@'%' IDENTIFIED BY '123456'"
openstack user create --domain default --password 123456 glance
openstack role add --project service --user glance admin
openstack service create --name glance --description "OpenStack Image" image
openstack endpoint create --region RegionOne image public http://controller:9292
openstack endpoint create --region RegionOne image internal http://controller:9292
openstack endpoint create --region RegionOne image admin http://controller:9292

yum install -y openstack-glance
sed -i.default -e '/^#/d' -e '/^$/d' /etc/glance/glance-api.conf
crudini --set /etc/glance/glance-api.conf database connection mysql+pymysql://glance:123456@controller/glance
crudini --set /etc/glance/glance-api.conf keystone_authtoken www_authenticate_uri http://controller:5000
crudini --set /etc/glance/glance-api.conf keystone_authtoken auth_url http://controller:5000
crudini --set /etc/glance/glance-api.conf keystone_authtoken memcached_servers controller:11211
crudini --set /etc/glance/glance-api.conf keystone_authtoken auth_type password
crudini --set /etc/glance/glance-api.conf keystone_authtoken project_domain_name Default
crudini --set /etc/glance/glance-api.conf keystone_authtoken user_domain_name Default
crudini --set /etc/glance/glance-api.conf keystone_authtoken project_name service
crudini --set /etc/glance/glance-api.conf keystone_authtoken username glance
crudini --set /etc/glance/glance-api.conf keystone_authtoken password 123456
crudini --set /etc/glance/glance-api.conf paste_deploy flavor keystone
crudini --set /etc/glance/glance-api.conf glance_store stores file,http
crudini --set /etc/glance/glance-api.conf glance_store default_store file
crudini --set /etc/glance/glance-api.conf glance_store filesystem_store_datadir /var/lib/glance/images
su -s /bin/sh -c "glance-manage db_sync" glance
systemctl enable openstack-glance-api && systemctl start openstack-glance-api

```

### 2.4 上传镜像

```shell
wget https://github.com/cirros-dev/cirros/releases/download/0.6.0/cirros-0.6.0-x86_64-disk.img
openstack image create "cirros" --file cirros-0.6.0-x86_64-disk.img --disk-format qcow2 --container-format bare --public
openstack image list
```

### 2.5 资源监控服务(Placement Service)

```shell
mysql -uroot -p123456 -e "CREATE DATABASE placement"
mysql -uroot -p123456 -e "GRANT ALL PRIVILEGES ON placement.* TO 'placement'@'localhost' IDENTIFIED BY '123456'"
mysql -uroot -p123456 -e "GRANT ALL PRIVILEGES ON placement.* TO 'placement'@'%' IDENTIFIED BY '123456'"
openstack user create --domain default --password 123456 placement
openstack role add --project service --user placement admin
openstack service create --name placement --description "Placement API" placement
openstack endpoint create --region RegionOne placement public http://controller:8778
openstack endpoint create --region RegionOne placement internal http://controller:8778
openstack endpoint create --region RegionOne placement admin http://controller:8778

yum install -y openstack-placement-api
sed -i.default -e '/^#/d' -e '/^$/d' /etc/placement/placement.conf
crudini --set /etc/placement/placement.conf placement_database connection mysql+pymysql://placement:123456@controller/placement
crudini --set /etc/placement/placement.conf api auth_strategy keystone
crudini --set /etc/placement/placement.conf keystone_authtoken auth_url http://controller:5000/v3
crudini --set /etc/placement/placement.conf keystone_authtoken memcached_servers controller:11211
crudini --set /etc/placement/placement.conf keystone_authtoken auth_type password
crudini --set /etc/placement/placement.conf keystone_authtoken project_domain_name Default
crudini --set /etc/placement/placement.conf keystone_authtoken user_domain_name Default
crudini --set /etc/placement/placement.conf keystone_authtoken project_name service
crudini --set /etc/placement/placement.conf keystone_authtoken username placement
crudini --set /etc/placement/placement.conf keystone_authtoken password 123456
su -s /bin/sh -c "placement-manage db sync" placement
systemctl restart httpd 
```

### 2.6 检查Placement安装结果

```shell
 placement-status upgrade check
 pip3 install osc-placement
 openstack --os-placement-api-version 1.2 resource class list --sort-column name
 openstack --os-placement-api-version 1.6 trait list --sort-column name


```

### 2.7 **管理UI界面Dashboard**

功能：web方式管理云平台，建云主机，分配网络，配安全组，加云盘

```shell
yum install -y openstack-dashboard
sed -i "39c ALLOWED_HOSTS = ['*']" /etc/openstack-dashboard/local_settings
sed -i "94c CACHES = {" /etc/openstack-dashboard/local_settings
sed -i "95c 'default': {" /etc/openstack-dashboard/local_settings
sed -i "96c 'BACKEND': 'django.core.cache.backends.memcached.MemcachedCache'," /etc/openstack-dashboard/local_settings
sed -i "97c 'LOCATION': 'controller:11211'," /etc/openstack-dashboard/local_settings
sed -i "98c }" /etc/openstack-dashboard/local_settings
sed -i "99c }" /etc/openstack-dashboard/local_settings
sed -i "104c SESSION_ENGINE = 'django.contrib.sessions.backends.cache'" /etc/openstack-dashboard/local_settings
sed -i '118c OPENSTACK_HOST = "controller"' /etc/openstack-dashboard/local_settings
sed -i '119c OPENSTACK_KEYSTONE_URL = "http://%s:5000/v3" % OPENSTACK_HOST' /etc/openstack-dashboard/local_settings
sed -i '123c TIME_ZONE = "Asia/Shanghai"' /etc/openstack-dashboard/local_settings

echo 'OPENSTACK_KEYSTONE_MULTIDOMAIN_SUPPORT = True' >> /etc/openstack-dashboard/local_settings
echo 'OPENSTACK_API_VERSIONS = {' >> /etc/openstack-dashboard/local_settings
echo '  "identity": 3,' >> /etc/openstack-dashboard/local_settings
echo '  "image": 2,' >> /etc/openstack-dashboard/local_settings
echo '  "volume": 3' >> /etc/openstack-dashboard/local_settings
echo '}' >> /etc/openstack-dashboard/local_settings
echo 'OPENSTACK_KEYSTONE_DEFAULT_DOMAIN = "default"' >> /etc/openstack-dashboard/local_settings
echo 'OPENSTACK_KEYSTONE_DEFAULT_ROLE = "user"' >> /etc/openstack-dashboard/local_settings

echo "WEBROOT = '/dashboard/'" >> /etc/openstack-dashboard/local_settings
echo 'WSGIApplicationGroup %{GLOBAL}' >> /etc/httpd/conf.d/openstack-dashboard.conf
systemctl restart httpd memcached


```

安装完成，可访问 http://10.0.0.10/dashboard/ 查看 用户名 admin 密码 123456

------

### 2.8 计算服务(Nova Service)

功能：负责响应虚拟机创建请求、调度、销毁云主

```shell
mysql -uroot -p123456 -e "CREATE DATABASE nova_api"
mysql -uroot -p123456 -e "CREATE DATABASE nova"
mysql -uroot -p123456 -e "CREATE DATABASE nova_cell0"
mysql -uroot -p123456 -e "GRANT ALL PRIVILEGES ON nova_api.* TO 'nova'@'localhost' IDENTIFIED BY '123456'"
mysql -uroot -p123456 -e "GRANT ALL PRIVILEGES ON nova_api.* TO 'nova'@'%' IDENTIFIED BY '123456'"
mysql -uroot -p123456 -e "GRANT ALL PRIVILEGES ON nova.* TO 'nova'@'localhost' IDENTIFIED BY '123456'"
mysql -uroot -p123456 -e "GRANT ALL PRIVILEGES ON nova.* TO 'nova'@'%' IDENTIFIED BY '123456'"
mysql -uroot -p123456 -e "GRANT ALL PRIVILEGES ON nova_cell0.* TO 'nova'@'localhost' IDENTIFIED BY '123456'"
mysql -uroot -p123456 -e "GRANT ALL PRIVILEGES ON nova_cell0.* TO 'nova'@'%' IDENTIFIED BY '123456'"
openstack user create --domain default --password 123456 nova
openstack role add --project service --user nova admin
openstack service create --name nova --description "OpenStack Compute" compute
openstack endpoint create --region RegionOne compute public http://controller:8774/v2.1
openstack endpoint create --region RegionOne compute internal http://controller:8774/v2.1
openstack endpoint create --region RegionOne compute admin http://controller:8774/v2.1
yum install -y openstack-nova-api openstack-nova-conductor openstack-nova-novncproxy openstack-nova-scheduler
sed -i.default -e '/^#/d' -e '/^$/d' /etc/nova/nova.conf
crudini --set /etc/nova/nova.conf DEFAULT enabled_apis osapi_compute,metadata
crudini --set /etc/nova/nova.conf DEFAULT transport_url rabbit://openstack:123456@controller:5672/
crudini --set /etc/nova/nova.conf DEFAULT my_ip 10.0.0.10
crudini --set /etc/nova/nova.conf api_database connection mysql+pymysql://nova:123456@controller/nova_api
crudini --set /etc/nova/nova.conf database connection mysql+pymysql://nova:123456@controller/nova
crudini --set /etc/nova/nova.conf api auth_strategy keystone
crudini --set /etc/nova/nova.conf keystone_authtoken www_authenticate_uri http://controller:5000/
crudini --set /etc/nova/nova.conf keystone_authtoken auth_url http://controller:5000/
crudini --set /etc/nova/nova.conf keystone_authtoken memcached_servers controller:11211
crudini --set /etc/nova/nova.conf keystone_authtoken auth_type password
crudini --set /etc/nova/nova.conf keystone_authtoken project_domain_name Default
crudini --set /etc/nova/nova.conf keystone_authtoken user_domain_name Default
crudini --set /etc/nova/nova.conf keystone_authtoken project_name service
crudini --set /etc/nova/nova.conf keystone_authtoken username nova
crudini --set /etc/nova/nova.conf keystone_authtoken password 123456
crudini --set /etc/nova/nova.conf vnc enabled true
crudini --set /etc/nova/nova.conf vnc server_listen '$my_ip'
crudini --set /etc/nova/nova.conf vnc server_proxyclient_address '$my_ip'
crudini --set /etc/nova/nova.conf glance api_servers http://controller:9292
crudini --set /etc/nova/nova.conf oslo_concurrency lock_path /var/lib/nova/tmp
crudini --set /etc/nova/nova.conf placement region_name RegionOne
crudini --set /etc/nova/nova.conf placement project_domain_name Default
crudini --set /etc/nova/nova.conf placement project_name service
crudini --set /etc/nova/nova.conf placement auth_type password
crudini --set /etc/nova/nova.conf placement user_domain_name Default
crudini --set /etc/nova/nova.conf placement auth_url http://controller:5000/v3
crudini --set /etc/nova/nova.conf placement username placement
crudini --set /etc/nova/nova.conf placement password 123456
crudini --set /etc/nova/nova.conf scheduler discover_hosts_in_cells_interval 300
su -s /bin/sh -c "nova-manage api_db sync" nova
su -s /bin/sh -c "nova-manage cell_v2 map_cell0" nova
su -s /bin/sh -c "nova-manage cell_v2 create_cell --name=cell1 --verbose" nova
su -s /bin/sh -c "nova-manage db sync" nova
systemctl enable openstack-nova-api openstack-nova-scheduler openstack-nova-conductor openstack-nova-novncproxy 
systemctl start openstack-nova-api openstack-nova-scheduler openstack-nova-conductor openstack-nova-novncproxy
```

### 2.9 网络服务neutron(Networking Service)

​	功能：实现SDN（软件定义网络），提供一整套API,用户可以基于该API实现自己定义专属网络，不同厂商可以基于此API提供自己的产品实现.

```shell
mysql -uroot -p123456 -e "CREATE DATABASE neutron"
mysql -uroot -p123456 -e "GRANT ALL PRIVILEGES ON neutron.* TO 'neutron'@'localhost' IDENTIFIED BY '123456'"
mysql -uroot -p123456 -e "GRANT ALL PRIVILEGES ON neutron.* TO 'neutron'@'%' IDENTIFIED BY '123456'"
openstack user create --domain default --password 123456 neutron
openstack role add --project service --user neutron admin
openstack service create --name neutron --description "OpenStack Networking" network
openstack endpoint create --region RegionOne network public http://controller:9696
openstack endpoint create --region RegionOne network internal http://controller:9696
openstack endpoint create --region RegionOne network admin http://controller:9696
yum install -y openstack-neutron openstack-neutron-ml2 openstack-neutron-linuxbridge ebtables

```

### 3.0配置服务器组件

```shell
sed -i.default -e '/^#/d' -e '/^$/d' /etc/neutron/neutron.conf
crudini --set /etc/neutron/neutron.conf DEFAULT core_plugin ml2
crudini --set /etc/neutron/neutron.conf DEFAULT service_plugins router
crudini --set /etc/neutron/neutron.conf DEFAULT allow_overlapping_ips true
crudini --set /etc/neutron/neutron.conf DEFAULT transport_url rabbit://openstack:123456@controller
crudini --set /etc/neutron/neutron.conf DEFAULT auth_strategy keystone
crudini --set /etc/neutron/neutron.conf DEFAULT notify_nova_on_port_status_changes true
crudini --set /etc/neutron/neutron.conf DEFAULT notify_nova_on_port_data_changes true
crudini --set /etc/neutron/neutron.conf database connection mysql+pymysql://neutron:123456@controller/neutron
crudini --set /etc/neutron/neutron.conf keystone_authtoken www_authenticate_uri http://controller:5000
crudini --set /etc/neutron/neutron.conf keystone_authtoken auth_url http://controller:5000
crudini --set /etc/neutron/neutron.conf keystone_authtoken memcached_servers controller:11211
crudini --set /etc/neutron/neutron.conf keystone_authtoken auth_type password
crudini --set /etc/neutron/neutron.conf keystone_authtoken project_domain_name default
crudini --set /etc/neutron/neutron.conf keystone_authtoken user_domain_name default
crudini --set /etc/neutron/neutron.conf keystone_authtoken project_name service
crudini --set /etc/neutron/neutron.conf keystone_authtoken username neutron
crudini --set /etc/neutron/neutron.conf keystone_authtoken password 123456
crudini --set /etc/neutron/neutron.conf oslo_concurrency lock_path /var/lib/neutron/tmp
crudini --set /etc/neutron/neutron.conf nova auth_url http://controller:5000
crudini --set /etc/neutron/neutron.conf nova auth_type password
crudini --set /etc/neutron/neutron.conf nova project_domain_name default
crudini --set /etc/neutron/neutron.conf nova user_domain_name default
crudini --set /etc/neutron/neutron.conf nova region_name RegionOne
crudini --set /etc/neutron/neutron.conf nova project_name service
crudini --set /etc/neutron/neutron.conf nova username nova
crudini --set /etc/neutron/neutron.conf nova password 123456


```

### 3.1 配置Modular Layer 2 (ML2) plug-in

```shell
crudini --set /etc/neutron/plugins/ml2/ml2_conf.ini ml2 type_drivers flat,vlan,vxlan
crudini --set /etc/neutron/plugins/ml2/ml2_conf.ini ml2 tenant_network_types vxlan
crudini --set /etc/neutron/plugins/ml2/ml2_conf.ini ml2 mechanism_drivers linuxbridge,l2population
crudini --set /etc/neutron/plugins/ml2/ml2_conf.ini ml2 extension_drivers port_security

crudini --set /etc/neutron/plugins/ml2/ml2_conf.ini ml2_type_flat flat_networks provider
crudini --set /etc/neutron/plugins/ml2/ml2_conf.ini ml2_type_vxlan vni_ranges 1:1000
crudini --set /etc/neutron/plugins/ml2/ml2_conf.ini securitygroup enable_ipset true
#配置Linux bridge agent
#eth1为另外第二张网卡
crudini --set /etc/neutron/plugins/ml2/linuxbridge_agent.ini linux_bridge physical_interface_mappings provider:ens33
crudini --set /etc/neutron/plugins/ml2/linuxbridge_agent.ini vxlan enable_vxlan true
#10.0.0.12为第二张网卡的IP
crudini --set /etc/neutron/plugins/ml2/linuxbridge_agent.ini vxlan local_ip 10.0.0.10
crudini --set /etc/neutron/plugins/ml2/linuxbridge_agent.ini vxlan l2_population true
crudini --set /etc/neutron/plugins/ml2/linuxbridge_agent.ini securitygroup enable_security_group true
crudini --set /etc/neutron/plugins/ml2/linuxbridge_agent.ini securitygroup firewall_driver neutron.agent.linux.iptables_firewall.IptablesFirewallDriver
#配置layer-3 agent
sed -i "2c interface_driver = linuxbridge" /etc/neutron/l3_agent.ini
#配置DHCP agent
sed -i.default -e '/^#/d' -e '/^$/d' /etc/neutron/dhcp_agent.ini
crudini --set /etc/neutron/dhcp_agent.ini DEFAULT interface_driver linuxbridge
crudini --set /etc/neutron/dhcp_agent.ini DEFAULT dhcp_driver neutron.agent.linux.dhcp.Dnsmasq
crudini --set /etc/neutron/dhcp_agent.ini DEFAULT enable_isolated_metadata true
#配置metadata agent
crudini --set /etc/neutron/metadata_agent.ini DEFAULT nova_metadata_host controller
crudini --set /etc/neutron/metadata_agent.ini DEFAULT metadata_proxy_shared_secret 123456
#配置计算服务使用网络服务
crudini --set /etc/nova/nova.conf neutron auth_url http://controller:5000
crudini --set /etc/nova/nova.conf neutron auth_type password
crudini --set /etc/nova/nova.conf neutron project_domain_name default
crudini --set /etc/nova/nova.conf neutron user_domain_name default
crudini --set /etc/nova/nova.conf neutron region_name RegionOne
crudini --set /etc/nova/nova.conf neutron project_name service
crudini --set /etc/nova/nova.conf neutron username neutron
crudini --set /etc/nova/nova.conf neutron password 123456
crudini --set /etc/nova/nova.conf neutron service_metadata_proxy true
crudini --set /etc/nova/nova.conf neutron metadata_proxy_shared_secret 123456
ln -s /etc/neutron/plugins/ml2/ml2_conf.ini /etc/neutron/plugin.ini
su -s /bin/sh -c "neutron-db-manage --config-file /etc/neutron/neutron.conf --config-file /etc/neutron/plugins/ml2/ml2_conf.ini upgrade head" neutron
systemctl restart openstack-nova-api
systemctl enable neutron-server neutron-linuxbridge-agent neutron-dhcp-agent neutron-metadata-agent 
systemctl start neutron-server neutron-linuxbridge-agent neutron-dhcp-agent neutron-metadata-agent
systemctl enable neutron-l3-agent && systemctl start neutron-l3-agent

```

验证

```shell
openstack network agent list
```

#### Linuxbridge 换 openvswitch

```shell
https://www.cnblogs.com/fengjian2016/p/16476413.html
```

### 3.2 块存储服务

```shell
mysql -uroot -p123456 -e "CREATE DATABASE cinder"

mysql -uroot -p123456 -e "GRANT ALL PRIVILEGES ON cinder.* TO 'cinder'@'localhost' IDENTIFIED BY '123456'"
mysql -uroot -p123456 -e "GRANT ALL PRIVILEGES ON cinder.* TO 'cinder'@'%' IDENTIFIED BY '123456'"
# 创建cinder用户
openstack user create --domain default --password-prompt cinder
# 将admin角色添加到cinder用户
openstack role add --project service --user cinder admin
# 创建cinderv2和cinderv3服务实体
openstack service create --name cinderv2 --description "OpenStack Block Storage" volumev2  
openstack service create --name cinderv3 --description "OpenStack Block Storage" volumev3
# 创建块存储服务API端点
openstack endpoint create --region RegionOne volumev2 public http://controller:8776/v2/%\(project_id\)s
openstack endpoint create --region RegionOne volumev2 internal http://controller:8776/v2/%\(project_id\)s
openstack endpoint create --region RegionOne volumev2 admin http://controller:8776/v2/%\(project_id\)s
openstack endpoint create --region RegionOne volumev3 public http://controller:8776/v3/%\(project_id\)s
openstack endpoint create --region RegionOne volumev3 internal http://controller:8776/v3/%\(project_id\)s
openstack endpoint create --region RegionOne volumev3 admin http://controller:8776/v3/%\(project_id\)s
yum -y install openstack-cinder
sed -i.default -e '/^#/d' -e '/^$/d' /etc/cinder/cinder.conf
crudini --set /etc/cinder/cinder.conf database connection mysql+pymysql://cinder:123456@controller/cinder
crudini --set /etc/cinder/cinder.conf DEFAULT transport_url rabbit://openstack:123456@controller
crudini --set /etc/cinder/cinder.conf DEFAULT auth_strategy keystone
crudini --set /etc/cinder/cinder.conf DEFAULT my_ip 10.0.0.10
crudini --set /etc/cinder/cinder.conf keystone_authtoken www_authenticate_uri http://controller:5000
crudini --set /etc/cinder/cinder.conf keystone_authtoken auth_url http://controller:5000
crudini --set /etc/cinder/cinder.conf keystone_authtoken memcached_servers controller:11211
crudini --set /etc/cinder/cinder.conf keystone_authtoken auth_type password
crudini --set /etc/cinder/cinder.conf keystone_authtoken project_domain_name default
crudini --set /etc/cinder/cinder.conf keystone_authtoken user_domain_name default
crudini --set /etc/cinder/cinder.conf keystone_authtoken project_name service
crudini --set /etc/cinder/cinder.conf keystone_authtoken username cinder
crudini --set /etc/cinder/cinder.conf keystone_authtoken password Mss123456
crudini --set /etc/cinder/cinder.conf oslo_concurrency lock_path /var/lib/cinder/tmp
# 同步数据库
su -s /bin/sh -c "cinder-manage db sync" cinder
# 计算配置为使用块存储
crudini --set /etc/nova/nova.conf cinder os_region_name RegionOne  
# 开机启动和启动服务
systemctl enable openstack-cinder-api openstack-cinder-scheduler
systemctl start openstack-cinder-api openstack-cinder-scheduler

```

## 二.计算节点

### 1.1 设置网卡

```shell
hostnamectl set-hostname compute-1
echo "10.0.0.10 controller" >> /etc/hosts
echo "10.0.0.11 compute-1" >> /etc/hosts

sed -i "s/BOOTPROTO=dhcp/BOOTPROTO=static/g" /etc/sysconfig/network-scripts/ifcfg-ens33
sed -i "s/ONBOOT=no/ONBOOT=yes/g" /etc/sysconfig/network-scripts/ifcfg-ens33
echo "IPADDR=10.0.0.11" >> /etc/sysconfig/network-scripts/ifcfg-ens33
echo "NETMASK=255.255.255.0" >> /etc/sysconfig/network-scripts/ifcfg-ens33
echo "GATEWAY=10.0.0.2" >> /etc/sysconfig/network-scripts/ifcfg-ens33
echo "DNS1=223.5.5.5" >> /etc/sysconfig/network-scripts/ifcfg-ens33
echo "DNS2=8.8.8.8" >> /etc/sysconfig/network-scripts/ifcfg-ens33
nmcli c reload
nmcli c up ens33

```

controller 为控制节点的域名，compute-1 为计算节点的域名

10.0.0.11为本地的IP址，255.255.255.0 为子网掩码，10.0.0.2 为网关地址， 223.5.5.5/223.6.6.6 这是阿里云提供的DNS服务器，8.8.8.8 这是google提供的DNS服务器。nmcli 为网络服务的控制命令(network 服务不是默认，已经弃用，可以安装)

设置DNS的方法除了在网卡设置上DNS1=x.x.x.x ，还可以编辑文件“/etc/resolv.conf” ，验证DNS 

```shell
echo "nameserver 223.5.5.5" >> /etc/resolv.conf
echo "nameserver 8.8.8.8" >> /etc/resolv.conf
```

### 1.2 验证DNs

```shell
dig alidns.com
```

### 1.3 **关闭swap分区，关闭SeLinux** 

```shell
swapoff -a
sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab
setenforce 0
sed -i 's#SELINUX=enforcing#SELINUX=disabled#g' /etc/selinux/config
systemctl disable firewalld && systemctl stop firewalld
```

### 1.4 时间服务

```shell
yum install -y chrony
sed -i '/^pool/d' /etc/chrony.conf
sed -i '/^server/d' /etc/chrony.conf
sed -i '2aserver controller iburst' /etc/chrony.conf
echo "pool ntp.aliyun.com iburst" >> /etc/chrony.conf
systemctl enable chronyd && systemctl start chronyd
timedatectl set-timezone Asia/Shanghai

```

ntp.aliyun.com 为阿里的NTP服务器，用于网络同步时间

### 1.5 基础组件

```shell
yum install -y openstack-release-wallaby 
yum install -y python3-openstackclient
#yum install -y python38
#ln -s /usr/bin/python3.8 /usr/bin/python
git clone https://github.com/pixelb/crudini.git
cd crudini
python3 setup.py install
#ln -s ~/crudini /usr/bin/crudini
#yum config-manager --set-enabled PowerTools
```

crudini的工具，在OpenEuler的源上是没有这个工具的，可以下载其源代码进行安装使用，本地应用使用python38进行安装使用。若是在CentOS8的数据源，可以直接安装，不需要下载源代码进行安装

```shell
yum install -y crudini
```

### 1.6 计算服务nova(Compute Service)

```shell
yum install -y openstack-nova-compute
sed -i.default -e '/^#/d' -e '/^$/d' /etc/nova/nova.conf
crudini --set /etc/nova/nova.conf DEFAULT enabled_apis osapi_compute,metadata
crudini --set /etc/nova/nova.conf DEFAULT transport_url rabbit://openstack:123456@controller
crudini --set /etc/nova/nova.conf DEFAULT my_ip 10.0.0.11
crudini --set /etc/nova/nova.conf api auth_strategy keystone
crudini --set /etc/nova/nova.conf keystone_authtoken www_authenticate_uri http://controller:5000/
crudini --set /etc/nova/nova.conf keystone_authtoken auth_url http://controller:5000/
crudini --set /etc/nova/nova.conf keystone_authtoken memcached_servers controller:11211
crudini --set /etc/nova/nova.conf keystone_authtoken auth_type password
crudini --set /etc/nova/nova.conf keystone_authtoken project_domain_name Default
crudini --set /etc/nova/nova.conf keystone_authtoken user_domain_name Default
crudini --set /etc/nova/nova.conf keystone_authtoken project_name service
crudini --set /etc/nova/nova.conf keystone_authtoken username nova
crudini --set /etc/nova/nova.conf keystone_authtoken password 123456
crudini --set /etc/nova/nova.conf vnc enabled true
crudini --set /etc/nova/nova.conf vnc server_listen 0.0.0.0
crudini --set /etc/nova/nova.conf vnc server_proxyclient_address '$my_ip' 
crudini --set /etc/nova/nova.conf vnc novncproxy_base_url http://controller:6080/vnc_auto.html
crudini --set /etc/nova/nova.conf glance api_servers http://controller:9292
crudini --set /etc/nova/nova.conf oslo_concurrency lock_path /var/lib/nova/tmp
crudini --set /etc/nova/nova.conf placement region_name RegionOne
crudini --set /etc/nova/nova.conf placement project_domain_name Default
crudini --set /etc/nova/nova.conf placement project_name service
crudini --set /etc/nova/nova.conf placement auth_type password
crudini --set /etc/nova/nova.conf placement user_domain_name Default
crudini --set /etc/nova/nova.conf placement auth_url http://controller:5000/v3
crudini --set /etc/nova/nova.conf placement username placement
crudini --set /etc/nova/nova.conf placement password 123456
crudini --set /etc/nova/nova.conf libvirt virt_type kvm
systemctl enable libvirtd openstack-nova-compute
systemctl start libvirtd openstack-nova-compute
```

\#执行命令，查看是否支持CPU虚拟化，如果大于0则支持

```shell
egrep -c '(vmx|svm)' /proc/cpuinfo
```

\#如果不支持的话还需要执行下面的命令

```shell
crudini --set /etc/nova/nova.conf libvirt virt_type qemu
systemctl restart libvirtd openstack-nova-compute
```

```shell
#OpenEuler 22.3版出现openstack-nova-compute启动不成功，如下解决
```

```shell
mkdir /usr/lib/python3.9/site-packages/instances
crudini --set /etc/nova/nova.conf DEFAULT compute_driver libvirt.LibvirtDriver
```

\#控制节点 计算节点从注册到发现会有延迟，根据discover_hosts_in_cells_interval 配置轮询发现时间，可以执行下面命令手动发现计算节点

```shell
su -s /bin/sh -c "nova-manage cell_v2 discover_hosts --verbose" nova
# 或者
openstack compute service list --service nova-compute
```

### 1.7 网络服务

```shell
yum install -y openstack-neutron-linuxbridge ebtables ipset
sed -i.default -e '/^#/d' -e '/^$/d' /etc/neutron/neutron.conf
crudini --set /etc/neutron/neutron.conf DEFAULT transport_url rabbit://openstack:123456@controller
crudini --set /etc/neutron/neutron.conf DEFAULT auth_strategy keystone
crudini --set /etc/neutron/neutron.conf keystone_authtoken www_authenticate_uri http://controller:5000
crudini --set /etc/neutron/neutron.conf keystone_authtoken auth_url http://controller:5000
crudini --set /etc/neutron/neutron.conf keystone_authtoken memcached_servers controller:11211
crudini --set /etc/neutron/neutron.conf keystone_authtoken auth_type password
crudini --set /etc/neutron/neutron.conf keystone_authtoken project_domain_name default
crudini --set /etc/neutron/neutron.conf keystone_authtoken user_domain_name default
crudini --set /etc/neutron/neutron.conf keystone_authtoken project_name service
crudini --set /etc/neutron/neutron.conf keystone_authtoken username neutron
crudini --set /etc/neutron/neutron.conf keystone_authtoken password 123456
crudini --set /etc/neutron/neutron.conf oslo_concurrency lock_path /var/lib/neutron/tmp
sed -i.default -e '/^#/d' -e '/^$/d' /etc/neutron/plugins/ml2/linuxbridge_agent.ini
# 自己的网卡
crudini --set /etc/neutron/plugins/ml2/linuxbridge_agent.ini linux_bridge physical_interface_mappings provider:ens33
crudini --set /etc/neutron/plugins/ml2/linuxbridge_agent.ini vxlan enable_vxlan true
# 自己的ip地址
crudini --set /etc/neutron/plugins/ml2/linuxbridge_agent.ini vxlan local_ip 10.0.0.11
crudini --set /etc/neutron/plugins/ml2/linuxbridge_agent.ini vxlan l2_population true
crudini --set /etc/neutron/plugins/ml2/linuxbridge_agent.ini securitygroup enable_security_group true
crudini --set /etc/neutron/plugins/ml2/linuxbridge_agent.ini securitygroup firewall_driver neutron.agent.linux.iptables_firewall.IptablesFirewallDriver
crudini --set /etc/neutron/plugins/ml2/linuxbridge_agent.ini vxlan l2_population true

#配置计算服务使用网络服务
crudini --set /etc/nova/nova.conf neutron auth_url http://controller:5000
crudini --set /etc/nova/nova.conf neutron auth_type password
crudini --set /etc/nova/nova.conf neutron project_domain_name default
crudini --set /etc/nova/nova.conf neutron user_domain_name default
crudini --set /etc/nova/nova.conf neutron region_name RegionOne
crudini --set /etc/nova/nova.conf neutron project_name service
crudini --set /etc/nova/nova.conf neutron username neutron
crudini --set /etc/nova/nova.conf neutron password 123456
systemctl restart openstack-nova-compute
systemctl enable neutron-linuxbridge-agent 
systemctl start neutron-linuxbridge-agent

```

## 三 .创建虚拟机

### 1.1 创建虚拟网络

```shell
# 1 创建网络
openstack network create  --share  --provider-physical-network provider --provider-network-type flat provider
 #2 查看
openstack network list 

```

### 1.2 创建子网

```shell
# 1 创建网络
openstack subnet create --network provider \
  --allocation-pool start=10.0.0.100,end=10.0.0.200 \
  --dns-nameserver 10.0.0.2 --gateway 10.0.0.2 \
  --subnet-range 10.0.0.0/24 provider-sub
# 2 查看
openstack subnet list
```

### 1.3 创建主机规格

```shell
#1 创建主机规格
openstack flavor create --id 0 --vcpus 1 --ram 64 --disk 1 m1.nano
# openstack flavor create 创建主机
# --id 主机ID
# --vcpus cpu数量
# --ram 64（默认是MB，可以写成G）
# --disk 磁盘（默认单位是G）
```

### 1.4 生成密钥对

```shell
# 1 生成密钥
ssh-keygen -q -N ""
# 2 上传密钥
openstack keypair create --public-key ~/.ssh/id_rsa.pub mykey
```

### 1.5 添加安全组规则

```shell
# 允许ICMP（ping）
openstack security group rule create --proto icmp default
# 允许安全shell（SSH）访问
openstack security group rule create --proto tcp --dst-port 22 default
```

### 1.6 查看创建实例需要的相关信息

```shell
openstack flavor list
openstack image list
openstack network list
openstack security group list
openstack keypair list
```

### 1.7 使用镜像创建虚拟机系统

```shell
openstack server create --flavor m1.nano --image cirros  --security-group default --key-name mykey hello-instance
#还可加入网络(net-id 在上一步获取)--nic net-id=9e07c3d5-9a9e-496c-90b6-ba294f8b0699
```

### 1.8 查看实例

```shell
openstack server list
```

### 1.9 登录

```shell
#ssh登录
ping 10.0.0.x
ssh cirros@10.0.0.x
#web登录的url
openstack console url show hello-instance
```


