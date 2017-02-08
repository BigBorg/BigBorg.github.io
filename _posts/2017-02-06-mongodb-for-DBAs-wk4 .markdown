---
layout: 	post
title:		"Mongdb for DBAs: Week4"
subtitle:	"Replication"
header-img:	"img/post-bg-deeplearning.jpg"
date:		2017-02-07
author: 	"Borg"
catalog:	true
tags:
    - Mongodb
---

# Replication

## Why replication?
- high availability (when server failure occurs)
- durability
- scaling in some situations
- disaster recovery

## Asynchronous vs syncronous replication
In asynchronous replication, data gets some time to transfer from primary to secondary.  
mongodb使用asynchronous replication, 数据从主数据库(primary)复制到从数据库(secondary)需要时间，如果主数据库离线时有部分写入数据未传输到从数据库，则当它再次上线的时候会回滚这些数据，从新的主数据库复制数据同步。

## selection of new primary
当主数据库离线时，剩下的数据库需要判断其是否真的离线，避免因为服务器间的通讯隔离导致的误判，原则是剩下的数据库进行投票达成一致，如果剩下的多数数据库（大于1/2）认为其已经离线则选出新的主数据库。

## replication commands
启动数据库时需要指定 **-replSet** 参数，相同的副本集的数据库需要指定相同的副本集名称。  
以下代码开启三个mongodb服务器进程，监听三个不同端口。

```shell
mkdir 1 2 3
mongod --dbpath 1 --port 27001 --replSet whatever
mongod --dbpath 2 --port 27002 --replSet whatever
mongod --dbpath 3 --port 27003 --replSet whatever
mongo --port 27001
```
mongo shell里的一些副本集指令：

```javascript
cfg = {
	"_id" : "whatever",
	"members" : [
		{
			"_id" : 0,
			"host" : "borg-TravelMate-P258-MG:27001"  # 此处禁止使用localhost，可以通过rs.status查看获取。
		},
		{
			"_id" : 1,
			"host" : "borg-TravelMate-P258-MG:27002"
		},
		{
			"_id" : 2,
			"host" : "borg-TravelMate-P258-MG:27003"
		}
	]
}
re.initiate(cfg)  # 初始化副本集
rs.add("borg-TravelMate-P258-MG:27002")  # 或者可以直接这样添加成员,不过仍然需要初始化，只是可以不提供配置。
rs.isMaster()
rs.stepDown()  # 主数据库下线，可以提供下线秒数，不能用于从数据库。
rs.reconfig(cfg)
rs.remove("borg-TravelMate-P258-MG:27002")

rs.slaveOk()
rs.syncFrom("borg-TravelMate-P258-MG:27001")
```
更多命令可直接用 rs.help() 获取。

## Read preference
读取数据时从哪个数据库读取有以下规则：

- Primary: 保证数据最新
- Primary preferred: 优先从主数据库读取，如果无法从主数据库获取则从从数据库获取。当主服务器下线而尚未选出新的主服务器时会有大概十秒间隔，这时候就可以从从数据库读取。缺点是当从从数据库读取时，数据可能不是最新。
- Secondary: 从从数据库读取，当需要对数据集进行分析报告之类消耗资源的计算时可以从一个从数据库上读取而不影响主数据库生产环境。
- Secondary preferred: 优先从从数据库读取。可以分散请求降低主数据库压力，但是当所有从数据库都突然下线时，所有请求涌入主数据库。
- Nearest: 最低延迟，当数据库分散各地时可以采用。
