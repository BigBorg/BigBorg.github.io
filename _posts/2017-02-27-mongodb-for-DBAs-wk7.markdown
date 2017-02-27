---
layout: 	post
title:		"Mongdb for DBAs: Week7"
subtitle:	"Backup and Recovery"
header-img:	"img/post-bg-deeplearning.jpg"
date:		2017-02-27
author: 	"Borg"
catalog:	true
tags:
    - Mongodb
---

# Security

- authentication
- access control/authorization
- encryption
- network setup
- auditting

## authentication
用户信息存储在**dbname.system.users**, 当dbname为admin时为全局设置，当使用数据库级别设置（即dbname不等于admin时）可以在不同的数据库使用相同的用户名而不产生冲突。

```javascript
mongod --auth
mongos --auth

use admin
db.system.users

var me = {user:"borg", pwd:"123", roles:["userAdminAnyDatabase"]}
db.createUser(me)

mongo -u borg -p 	# 连接时验证
db.auth(name,password)  # 连接后再验证
```

用户的角色可以是：

- read
- readWrite
- dbAdmin
- userAdmin
- clusterAdmin
- readAnyDatabase
- readWriteAnyDatabase
- dbAdminAnyDatabase
- userAdminAnyDatabase

## Intra-cluster Security
--keyfile用于mongod进程间的验证，mongos也需要使用。

```shell
touch keyfile
chmod 600 keyfile
openssl rand -base64 60 >> keyfile
mongod --keyFile keyfile
mongos --keyFile keyfiel
```

## Backing Up
- mongodump
- filesystem snapshot
- backup from secondary
  - shutdown, copyfiles, restart

### mongodump
mongodump --oplog
mongorestore --oplogReplay

### filesystem snapshot
journaling should be enabled

### backing up a sharded cluster
1. turn off balancer: sh.stopBalancer()
2. backup config db: mongodump --db config
3. back up each shard's ReplSet
4. sh.startBalancer()

# Additional Features of MongoDB
- capped collection:	document in capped database can not be reallocated(delete or grow)
- TTL collection
- GridFS:	for data larger than 16MB

# Additional Resources
- **docs**:  mongodb.org
- **driver docs**
- jira.mongodb.org
- support forums: mongodb-user in google groups
- IRC: freenode.net/#mongodb
- github.com for source code
- blog.mongodb.org
- meetup.com
