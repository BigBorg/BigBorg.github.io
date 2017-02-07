---
layout: 	post
title:		"Mongdb for Developers: Week4"
subtitle:	"Performance"
header-img:	"img/post-bg-deeplearning.jpg"
date:		2017-02-06
author: 	"Borg"
catalog:	true
tags:
    - Mongodb
---

# Performance
本周的内容基本与上周[Mongodb for DBAs: week3](https://bigborg.github.io/2017/01/30/mongodb-for-DBAs-wk3/)重复，重点和之前没有的再记一次好了。

## MMAPv1 vs wiredTiger
相比于MMAPv1, wiredTiger支持
- 文档级别锁，准确的说不叫锁，英文叫**document level concurrency**。
- 压缩，wiredTiger自己决定内存的时候，哪些页需要从磁盘读出到内存哪些需要写入磁盘，因此可以压缩后再写入。因为文档的key基本会大量重复，所以压缩还是蛮有用的。
不支持
- **in place update** 对于内存中某个文档的修改，wiredTiger会将该文档弃用而新开辟内存放新的文档。这个MMAPv1是支持的。

## Multikey Indexes(区分multikey 和 compound key)
对于索引 {a:1,b:1} 插入的文档a和b不能同时为列表，但其中一个可以是列表。

## When is an index used?
eg. 索引a_1_b_1_c_1, 以下会使用到索引，注意最后两种情形

```javascript
db.collection.find({a:3})
db.collection.find({a:3,b:4})
db.collection.find({a:3,b:4,c:5})
db.collection.find({a:3,c:5})
db.collection.find({c:3}).sort({a:1,b:1})
```

## Index Size
db.collection.stats()
db.collection.indexSize()

## Index Design Rule
order: Equality fields, sort fields, range fields

## mongotop & mongostat
mongotop 3 每隔3秒输出一次每个集合读写花费时间
mongostat
