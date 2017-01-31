---
layout: 	post
title:		"Mongdb for DBAs: Week3"
subtitle:	"Performance"
header-img:	"img/post-bg-deeplearning.jpg"
date:		2017-01-30
author: 	"Borg"
catalog:	true
tags:
    - Mongodb
---

# Storage Engine
Mongodb 现有两个存储引擎，默认的是MMAPv1，可选的是WiredTiger。可以在启动数据库时指定。wiredTiger支持而MMAPv1不支持的特性有：文档级别锁，数据压缩。
```shell
mongod --storageEngine wiredTiger
```

具体存储的结构无图不好说，推荐阅读：[MongoDB的存储结构及对空间使用率的影响](http://www.mongoing.com/blog/file-storage)。值得说的一点是文档空间的分配是按照2的次方分配的，可以减小文件修改导致的文件移动次数，跟数据结构那本教材里的内存管理有点相通。如果一个原本较小的文档但能够预料到它会被更新得较大，则可以在插入文档的时候创建一个placehoder字符串预先占用以在创建时就分配较大的空间。

# Index
## createIndex, getIndexes & dropIndex

```javascript
db.collection.createIndex({field1:1, field2:1}) # compound index, 1代表升序
db.collection.getIndexes()
db.collection.dropIndex({field1:1, field2:1})
db.collection.dropIndex("indexname")
```
创建索引的时候可以设定升序降序，上面的代码中的例子按照.sort({field1:1,field2:1})或者.sort({field1:-1,field2:-1})都会使用到索引，但不能是.sort({field1:1, field2:-1})或者.sort({field1:-1,field2:1})

## Index note
- 索引不在乎值是什么类型，被索引的key在一个集合内（不同文档）可以同时有数值，null，字符串多种类型。
- {likes:["tennis", "golf"]} 列表可以被索引，讲对列表内每一个元素创建索引，即对tennis和golf分别创建索引。
- 可以索引内嵌文档和内嵌文档内的key。如： {wind:{directioin: "N", speed:12.2}} .ensureIndex({wind:1})索引整个内嵌文档，用于比较整个内嵌文档是否相等。.ensureIndex({"wind.direction":1, "wind.speed":1})索引内嵌key。

## Unique index
唯一索引，可以对单一键索引也可以使组合索引。_id 字段就是unique index。当key被索引为unique时，不能有重复值，包括空，即**不能有两个文档都不具有该字段**，如果集合内只有少数文档具有该字段，则可以设置sparse选项为true即可同有多个文档都不具有该字段。
```javascript
db.collection.ensureIndex({key1:1, key2:1}, {unique:true, sparse:false})
```

## TTL
Time To Live 即在多少时间段之后删除文档的索引，通过**expireAfterSeconds选项设置。值得说明的是下方的代码并不会在精确的一小时后删除插入的文档，而是过了一小时这个时间点的某个时间会被数据库清除。

```javascript
db.collection.createIndex({key:1},{expireAfterSeconds:3600})
```

## Geospatial index
地理位置索引，或者说是对二维数字索引。2dsphere用于地球上的位置，因为坐标系不是平面的xy坐标。而例如游戏这样的应用如果模拟平面的世界则可以使用2d索引。  
下方代码中的$geometry值其实是用json记录地理位置的一项标准，并不是mognodb自创的。参照[geojson文档](http://geojson.org/geojson-spec.html#geometry-objects)。  
地理位置支持的操作符参考[mongodb geospatical operators](https://docs.mongodb.com/manual/reference/operator/query-geospatial/)

```javascript
{loc: [2,4]}
db.collection.ensureIndex({loc:"2dsphere"})
db.collection.ensureIndex({loc:"2d"})
db.collection.find({loc: {$near: {$geometry: {type:"Point", coordinates:[2, 2.2]}, spherical: true}}})
db.collection.find({loc: {$geoWithin: {$geometry: {type:"Polygon", coordinates:[[ # 两个方括号是为了支持内部带有洞的多边形
	[0,0], [1,1], [2,2], [0,0]
]]}, spherical:true}}})
```

## Text index
使用正则表达式搜索效率低，应用文本索引实现搜索引擎。原理同列表索引相似，即对文本内每个单词进行索引。注意查找的query中并不包含搜索的字段名，还有匹配程度的使用。

```javascript
db.collection.createIndex({field: "text"})
db.collection.find({$text: {$search: "keyword1 keyword2"}}, {score:{$meta:"textScore"}}).sort({score:{$meta:"textScore"}}) # 按照匹配程度排序
```

## Background index creation
索引创建在primary可以在后台运行，secondary则不能。后台运行相对较慢，但创建过程中集合仍可读写。前台运行更适合较大数据集，效率更高，更高压缩。

```javascript
db.collection.ensureIndex({field:1},{background:true]) # 该行代码不会立即返回，虽然是后台运行，该行代码仍需要等待，后台指的此时其它连接仍可对集合读写。
```

# Explain Plans

```javascript
db.collection.explain().find({...})  # 不会真正运行查找，查看会使用的索引
db.collection.explain("executionStats").find({...})  # 实际运行，获取具体数据，如扫描的文档数，用时等
db.collection.explain("allPlansExecution").find({...})  # 选出效率最佳的索引
```

# Covered Query
最近运行过的查找再次查找时不需要搜索文档

# Read & Write Recap 
索引会增加插入的时间，但会减少查找的时间。导入数据时，先导入数据再创建索引比先创建索引再导入数据快。

# db.currentOp & db.killOp
可以安全kill的操作包括： find, findAndModify, foreground index creation on primary
不可以安全地kill的操作： foregournd index creation on secondary（会导致primary有索引，secondary没有，不一致）, a compact command job（数据未损失，但索引会出错）

# profiler
db level，设置一个数据库的profile level不会影响其它数据库。存于system.profile集合

```javascript
db.setProfilingLevel(2)    # 0 for off, 1 for selective, 2 for on
db.setProfilingLevel(1, 3)  # only operations that takes longer than 3 milliseconds will be recorded
```
