---
layout: 	post
title:		"Mongodb for DBAs: Week2"
subtitle:	"CRUD & administrative commands"
header-img:	"img/post-bg-deeplearning.jpg"
date:		2017-01-25
author: 	"Borg"
catalog:	true
tags:
    - mongodb
---

# CRUD
CRUD因在另一门课[Mongodb for Developers: Week2](http://bigborg.github.io/2017/01/23/mongodb-wk2-forDevelopers/)笔记中基本覆盖，所以重复内容不再做笔记。也可参考[官方文档](https://docs.mongodb.com/manual/crud/#create-operations)。

## Bulk() operations and methods

```javascript
var bulk = db.items.initializeUnorderedBulkOp();
bulk.insert({item:1})
bulk.insert({item:2})
bulk.insert({item:3})
bulk.execute()  # 这步时上面的insert才作为一次请求发送给数据库
```
bulk可以进行除了insert外的操作，如update/delete，同样需要execute才会写入数据库，但不能execute多次。

# Commands
[文档](https://docs.mongodb.com/manual/reference/command/)  
db.runCommand({ command }) 调用命令

```javascript
db.runCommand({isMaster:1})
db.runCommand("isMaster")
db.isMaster()
db.serverStatus()
db.currentOp()
db.killOp(opid)
db.collection.stats()
db.collection.drop()
```

- server level: isMaster, serverStatus, logout, getLastError
- db level: dropDatabase, repairDatabase, clone, copydb, dbStats
- collection level: 
  - DBA: create, drop, collStats, renameCollection
  - User: count, aggregate, mapreduce, findAndModify, geo*
- index level: ensureIndex, dropIndex
