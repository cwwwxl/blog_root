---
title: openstack服务相关命令
cover: 'https://s2.loli.net/2023/01/26/28KtaLZVC5Nkpue.png'
tags: openstack
categories: 云计算
top_img: 'https://s2.loli.net/2023/01/26/2duW1cJBnCgkwPo.png'
abbrlink: f612fab2
sticky:
---
## openstack服务相关命令

```shell
重启openstack的整个服务
openstack-service restart

1. 重启dashboard
service httpd restart
service memcached restart

2. 重启 ceilometer
2.1 cinder
service mongod restart
2.2 controller
service openstack-ceilometer-api restart
service openstack-ceilometer-notification restart
service openstack-ceilometer-central restart
service openstack-ceilometer-collector restart
service openstack-ceilometer-alarm-evaluator restart
service openstack-ceilometer-alarm-notifier restart
2.3 compute
service openstack-nova-compute restart
2.4 controller
service openstack-glance-api restart
service openstack-glance-registry restart
Block Storage service
2.5 controller node
service openstack-cinder-api restart
service openstack-cinder-scheduler restart
2.6 cinder
service openstack-cinder-volume restart

3. 重启Fuel服务
docker restart fuel-core-6.1-nailgun
docker restart fuel-core-6.1-keystone
docker restart fuel-core-6.1-rsync
docker restart fuel-core-6.1-mcollective
docker restart fuel-core-6.1-ostf
docker restart fuel-core-6.1-astute
docker restart fuel-core-6.1-rsyslog
docker restart fuel-core-6.1-postgres
docker restart fuel-core-6.1-rabbitmq
docker restart fuel-core-6.1-nginx
docker restart fuel-core-6.1-cobbler

4. 重启 Neutron 服务
4.1 控制节点
service openstack-nova-api restart
service openstack-nova-scheduler restart
service openstack-nova-conductor restart
service neutron-server restart
4.2 网络节点
service openvswitch restart
service neutron-openvswitch-agent restart（fuel控制节点默认stop）
service neutron-l3-agent restart（fuel控制节点默认stop）
service neutron-dhcp-agent restart（fuel控制节点默认stop）
service neutron-metadata-agent restart（fuel控制节点默认stop）
4.3 计算节点
service neutron-openvswitch-agent restart
service openvswitch restart

5. 重启cinder服务
5.1 控制节点
service openstack-cinder-api restart
service openstack-cinder-scheduler restart
5.2 存储节点
service openstack-cinder-volume restart

6. 重启glance服务
6.1 控制节点
service openstack-glance-api restart
service openstack-glance-registry restart

7. 重启Swift服务
7.1 控制节点
service openstack-swift-proxy restart
service memcached restart
7.2 存储节点
service openstack-swift-account restart
service openstack-swift-account-auditor restart
service openstack-swift-account-reaper restart
service openstack-swift-account-replicator restart
service openstack-swift-container restart
service openstack-swift-container-auditor restart
service openstack-swift-container-replicator restart
service openstack-swift-container-updater restart
service openstack-swift-object restart
service openstack-swift-object-auditor restart
service openstack-swift-object-replicator restart
service openstack-swift-object-updater restart

8. 重启Nova服务
8.1 控制节点
service openstack-nova-api restart

or:/etc/init.d/openstack-nova-api restart

service openstack-nova-cert restart
service openstack-nova-consoleauth restart
service openstack-nova-scheduler restart
service openstack-nova-conductor restart
service openstack-nova-novncproxy restart
8.2 计算节点
service libvirtd restart
service openstack-nova-compute restart
```

