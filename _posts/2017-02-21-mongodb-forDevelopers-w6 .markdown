---
layout: 	post
title:		"Mongdb for Developers: Week6"
subtitle:	"Application Engine"
header-img:	"img/post-bg-deeplearning.jpg"
date:		2017-02-21
author: 	"Borg"
catalog:	true
tags:
    - Mongodb
---

# Connecting to a Replica Set from Pymongo
```py
read_pref = pymongo.read_preferences.ReadPreference.SECONDARY
connection = pymongo.MongoClient()  # for default localhost:port
connection = pymongo.MongoClient(host=['mongodb://localhost:27001', 'mongodb://localhost:27002'], replicaSet="setName", w=1, j=False, read_preference = read_pref)
```

host参数只要列出可用的一个node即可，pymongo会自动查找到其余的成员。  
**w** 设置保证成功写入的副本个数。可以为数字也可以为"majority"  
**j** 设置是否等待（Primary）日志写入。  
**read_preference** One of primary, primaryPreferred, secondary, secondaryPreferred, or nearest. Defaults to primary.

# Catching Failover
Primary离线时，pymongo会抛出pymong.errors.AutoReconnect异常，捕获异常等待几秒重试，会自动连接到其它节点。但抛出AutoReconnect异常时无法保证数据成功写入磁盘也无法保证数据未写入，所以如果再次插入带相同_id字段文档有可能会抛出pymongo.errors.DuplicateKeyError。

```py
for retry in range(3):
    try:
        # insert something
        break
    except pymongo.errors.AutoReconnect as e:
        print(e)
        time.sleep(5)
    except pymong.errors.DuplicateKeyError as e:    # 如果是读取则不需要捕获DuplicateKeyError
        print(e)
        break
```

对于update，**$inc、$push** 不应该被重复执行，所以重试的时候需要验证上次操作是否成功，或者使用 **$set** 则无需验证。

# Sharding
sharding 用于把一个较大的集合拆分存储。拆分的方法有**range based**和**hash based**。

## Building a Sharded Environment
以下是mongodb university公开课提供的shell代码，创建shards，注意 --shardsvr --shardsvr 选项和 mongos 命令。创建2个shards，每个shard包含3个replicate set，则一共创建需要9个mognod进程。

```bash
# Andrew Erlichson
# MongoDB
# script to start a sharded environment on localhost

# clean everything up
echo "killing mongod and mongos"
killall mongod
killall mongos
echo "removing data files"
rm -rf /data/config
rm -rf /data/shard*


# start a replica set and tell it that it will be shard0
echo "starting servers for shard 0"
mkdir -p /data/shard0/rs0 /data/shard0/rs1 /data/shard0/rs2
mongod --replSet s0 --logpath "s0-r0.log" --dbpath /data/shard0/rs0 --port 37017 --fork --shardsvr --smallfiles
mongod --replSet s0 --logpath "s0-r1.log" --dbpath /data/shard0/rs1 --port 37018 --fork --shardsvr --smallfiles
mongod --replSet s0 --logpath "s0-r2.log" --dbpath /data/shard0/rs2 --port 37019 --fork --shardsvr --smallfiles

sleep 5
# connect to one server and initiate the set
echo "Configuring s0 replica set"
mongo --port 37017 << 'EOF'
config = { _id: "s0", members:[
          { _id : 0, host : "localhost:37017" },
          { _id : 1, host : "localhost:37018" },
          { _id : 2, host : "localhost:37019" }]};
rs.initiate(config)
EOF

# start a replicate set and tell it that it will be a shard1
echo "starting servers for shard 1"
mkdir -p /data/shard1/rs0 /data/shard1/rs1 /data/shard1/rs2
mongod --replSet s1 --logpath "s1-r0.log" --dbpath /data/shard1/rs0 --port 47017 --fork --shardsvr --smallfiles
mongod --replSet s1 --logpath "s1-r1.log" --dbpath /data/shard1/rs1 --port 47018 --fork --shardsvr --smallfiles
mongod --replSet s1 --logpath "s1-r2.log" --dbpath /data/shard1/rs2 --port 47019 --fork --shardsvr --smallfiles

sleep 5

echo "Configuring s1 replica set"
mongo --port 47017 << 'EOF'
config = { _id: "s1", members:[
          { _id : 0, host : "localhost:47017" },
          { _id : 1, host : "localhost:47018" },
          { _id : 2, host : "localhost:47019" }]};
rs.initiate(config)
EOF

# start a replicate set and tell it that it will be a shard2
echo "starting servers for shard 2"
mkdir -p /data/shard2/rs0 /data/shard2/rs1 /data/shard2/rs2
mongod --replSet s2 --logpath "s2-r0.log" --dbpath /data/shard2/rs0 --port 57017 --fork --shardsvr --smallfiles
mongod --replSet s2 --logpath "s2-r1.log" --dbpath /data/shard2/rs1 --port 57018 --fork --shardsvr --smallfiles
mongod --replSet s2 --logpath "s2-r2.log" --dbpath /data/shard2/rs2 --port 57019 --fork --shardsvr --smallfiles

sleep 5

echo "Configuring s2 replica set"
mongo --port 57017 << 'EOF'
config = { _id: "s2", members:[
          { _id : 0, host : "localhost:57017" },
          { _id : 1, host : "localhost:57018" },
          { _id : 2, host : "localhost:57019" }]};
rs.initiate(config)
EOF


# now start 3 config servers
echo "Starting config servers"
mkdir -p /data/config/config-a /data/config/config-b /data/config/config-c 
mongod --logpath "cfg-a.log" --dbpath /data/config/config-a --port 57040 --fork --configsvr --smallfiles
mongod --logpath "cfg-b.log" --dbpath /data/config/config-b --port 57041 --fork --configsvr --smallfiles
mongod --logpath "cfg-c.log" --dbpath /data/config/config-c --port 57042 --fork --configsvr --smallfiles


# now start the mongos on a standard port
mongos --logpath "mongos-1.log" --configdb localhost:57040,localhost:57041,localhost:57042 --fork
echo "Waiting 60 seconds for the replica sets to fully come online"
sleep 60
echo "Connnecting to mongos and enabling sharding"

# add shards and enable sharding on the test db
mongo <<'EOF'
db.adminCommand( { addshard : "s0/"+"localhost:37017" } );
db.adminCommand( { addshard : "s1/"+"localhost:47017" } );
db.adminCommand( { addshard : "s2/"+"localhost:57017" } );
db.adminCommand({enableSharding: "school"})
db.adminCommand({shardCollection: "school.students", key: {student_id:1}});
EOF
```
