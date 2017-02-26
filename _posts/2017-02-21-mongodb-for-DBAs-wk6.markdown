---
layout: 	post
title:		"Mongdb for DBAs: Week6"
subtitle:	"Scalability"
header-img:	"img/post-bg-deeplearning.jpg"
date:		2017-02-21
author: 	"Borg"
catalog:	true
tags:
    - Mongodb
---

# Sharding & Data Distribution
shard key 用于决定文档存于哪个 shard, 分为 **range based** 和 **hash based**。以 range based 为例，一个[min, max]范围形成一个chunk，取一个中间的数字将原来的区块分为两个（splite），再把新分出来的区块发送（migration）给其它shard。 

# Running
需要三种进程：

- --shardsvr   实际存储数据的进程
- --configsvr  一般三个足够
- mongos --configdb host:port,host1:port1,host3:port3  最好在 27017 默认端口上运行，任意数量，但一般大于1，configdb参数后为configsvr配置服务器地址。

各个进程创建好，初始化replicate set后，连接mongos添加shard，再设置shard key。
如果未设置shard key，则数据默认存储在primary shard，不会分配到其它shard。

```js
sh.addShard("host:port")  # 只要添加每个shard内replicate set里的primary
sh.enableSharding(dbname)
sh.shardCollection("dbname.collection",key,unique)  # unique 为bool，可选参数
sh.shardCollection("collectionname",{shardkey:1},true)
```

# Shard Key Selection
- sufficient cardinality:  enough values to spread data out
- consider compound shard keys
- Is the key monotonically increasing?  # 如果sharkey单调增长，最后一个shard将会存储过多数据。注： BSON ObjectId 即 _id 字段会单调增长

# Process and Machine Layout
shard以及底下的replicate set应该分散在不同的机器上，而conifg server可以与任意replicate set运行在同一机器上，mongos也可以与replicate set运行在同一机器上，也可以运行在用户的客户端。举例：3 shards, 3 replicate sets each shard, 3 config server and 6 mongos process might use 9 machines.  
注意replicate set数量最好是奇数。

# Bulk Inserts and Pre-splitting
bulk insert时，会先写入到其中一个shard，再执行split与migrate，如果插入数据的速度大于migrate的速度，这时就应该考虑使用pre-splitting：  
**sh.splitAt(collection, {key:val})**  
预先添加分割点

# Tips and Best Practices
- only shard big collections
- pick shard key carefully (can not be changed once set)
- consider presplitting on a bulk load
- be aware of monotonically increasing shard key values on inserts
- adding shards is fairly easy but isn't instaneous
- always connet to mongos except for some DBA work: put mongos on default port
- use logical config server names

