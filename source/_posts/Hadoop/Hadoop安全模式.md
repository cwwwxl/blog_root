---
title: hadoop安全模式
cover: 'https://s2.loli.net/2023/03/24/c6JCiWt3Yo1lqdF.jpg'
tags: Hadoop
categories: Hadoop
top_img: 'https://s2.loli.net/2023/03/24/c6JCiWt3Yo1lqdF.jpg'
abbrlink: 61691ff1
---
# Hadoop安全模式

#### 1.什么是安全模式

​	安全模式是HDFS所处的一种特殊状态，在这种状态下，文件系统只接受读数据请求，而不接受删除、修改等变更请求。

​	在NameNode主节点启动时，HDFS首先进入安全模式，DataNode在启动的时候会向namenode汇报可用的block等状态，当整个系统达到安全标准时，HDFS自动离开安全模式。如果HDFS出于安全模式下，则文件block不能进行任何的副本复制操作，因此达到最小的副本数量要求是基于datanode启动时的状态来判定的，启动时不会再做任何复制（从而达到最小副本数量要求）。

​	如果没有安全模式，Hadoop集群使用的风险较大。集群的安全机制是较大的，Hadoop的开发者就设定了一个安全模式，说明如下：

- 用户只能访问有权限的的HDFS目录或文件。
- 用户只能访问或修改自身的Mapreduce任务。
- 用户使用Hadoop集群的相关服务要进行身份认证。
- 服务和服务之间也需要相互认证，以防止未经授权的服务。

#### 3 相关命令

```shell
hadoop dfsadmin -safemode <command>

- get 查看当前状态
- enter 进入安全模式
- leave  强制离开安全模式
- wait 一直等待直到安全模式结束
```



### Overview 'master0:8020' (active)

| Started:       | Sat Mar 18 20:10:30 +0800 2023                      |
| :------------- | --------------------------------------------------- |
| Version:       | 3.3.0, rUnknown                                     |
| Compiled:      | Thu Jul 15 15:35:00 +0800 2021 by root from Unknown |
| Cluster ID:    | CID-669e7490-b65b-422c-b7be-3604c7ab3c41            |
| Block Pool ID: | BP-1833732125-192.168.169.110-1678865312411         |

### Summary

Security is off.

Safemode is off.

1 files and directories, 0 blocks (0 replicated blocks, 0 erasure coded block groups) = 1 total filesystem object(s).

Heap Memory used 99.39 MB of 220 MB Heap Memory. Max Heap Memory is 1.02 GB.

Non Heap Memory used 54.61 MB of 56 MB Commited Non Heap Memory. Max Non Heap Memory is <unbounded>.

| Configured Capacity:                                         | 45.22 GB                                 |
| :----------------------------------------------------------- | ---------------------------------------- |
| Configured Remote Capacity:                                  | 0 B                                      |
| DFS Used:                                                    | 8 KB (0%)                                |
| Non DFS Used:                                                | 2.95 GB                                  |
| DFS Remaining:                                               | 42.27 GB (93.49%)                        |
| Block Pool Used:                                             | 8 KB (0%)                                |
| DataNodes usages% (Min/Median/Max/stdDev):                   | 0.00% / 0.00% / 0.00% / 0.00%            |
| [Live Nodes](http://192.168.169.110:9870/dfshealth.html#tab-datanode) | 1 (Decommissioned: 0, In Maintenance: 0) |
| [Dead Nodes](http://192.168.169.110:9870/dfshealth.html#tab-datanode) | 0 (Decommissioned: 0, In Maintenance: 0) |
| [Decommissioning Nodes](http://192.168.169.110:9870/dfshealth.html#tab-datanode) | 0                                        |
| [Entering Maintenance Nodes](http://192.168.169.110:9870/dfshealth.html#tab-datanode) | 0                                        |
| [Total Datanode Volume Failures](http://192.168.169.110:9870/dfshealth.html#tab-datanode-volume-failures) | 0 (0 B)                                  |
| Number of Under-Replicated Blocks                            | 0                                        |
| Number of Blocks Pending Deletion (including replicas)       | 0                                        |
| Block Deletion Start Time                                    | Sat Mar 18 20:10:30 +0800 2023           |
| Last Checkpoint Time                                         | Sat Mar 18 20:10:46 +0800 2023           |
| Enabled Erasure Coding Policies                              | RS-6-3-1024k                             |

- Configured Capacity: 表示已经配置的文件系统存储总量，目前为45.22 GB
- DFS Used: 表示已经使用的HDFS存储总量，目前  8 KB (0%)
- Non DFS Used:表示被非HDFS的应用所占用的存储总量，目前为2.95 GB
- DFS Remaining:表示HDFS可使用的存储总量，42.27 GB (93.49%)
- Live Nodes：表示在线的DataNode，目前是一个



### NameNode Journal Status

**Current transaction ID:** 8

| Journal Manager                                             | State                                                        |
| :---------------------------------------------------------- | :----------------------------------------------------------- |
| FileJournalManager(root=/export/data/hadoop-3.3.0/dfs/name) | EditLogFileOutputStream(/export/data/hadoop-3.3.0/dfs/name/current/edits_inprogress_0000000000000000008) |

### NameNode Storage

| **Storage Directory**              | **Type**        | **State** |
| ---------------------------------- | --------------- | --------- |
| /export/data/hadoop-3.3.0/dfs/name | IMAGE_AND_EDITS | Active    |

### DFS Storage Types

Storage TypeConfigured CapacityCapacity UsedCapacity RemainingBlock Pool UsedNodes In ServiceDISK45.22 GB8 KB (0%)42.27 GB (93.49%)8 KB1