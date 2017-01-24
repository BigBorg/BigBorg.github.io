---
layout: 	post
title:		"Mongodb for Developers: Week2"
subtitle:	"Mongodb CRUD"
header-img:	"img/post-bg-deeplearning.jpg"
date:		2017-01-23
author: 	"Borg"
catalog:	true
tags:
    - Mongodb
---
# Mongodb University Courses Note
Mongodb University的M101P: MongoDB for Developers与M102: MongoDB for DBAs课程第一周笔记  
[官方文档地址](https://docs.mongodb.com/v3.2/)  
[api文档](http://api.mongodb.com)

# CRUD

## Create
```javascript
db.collection.insertOne()
db.collection.insertMany([ {...},{...} ], {"ordered":false})  # 详细参数见文档，第二个参数可省。Excluding Write Concern errors, ordered operations stop after an error, while unordered operations continue to process any remaining write operations in the queue.
```

## Retrieve
```javascript
db.collection.find({"actors":"Mark Fergus"}) # 查找的结果中actors字段可以为列表，只要包含Mark Fergus即可( Match any element of an array)
db.collection.find({"actors.0":"Mark Fergus"}) # match the first element of an array
db.collection.find( {"actors":["Mark Fergus", "Jeff Bridges"]} ) # 查找的结果中actors字段必须包含Mark和Jeff，并且Mark在Jeff前。
```

### cursors
游标batch中剩余的文档数

```javascript
cur.objsLeftInBatch()
```

### projection
只返回需要的字段，Week1已讲，db.collection.find() 的第二个参数实现。

```javascript
db.movies.find({}, {field1:1,_id:0})
```

### Operators

#### Comparison Operators
$eq, $gt, $gte, $lt, $lte, $ne, $in, $nin

```javascript
db.collection.find({"field":{"$lt":100,"$gt":60}})
```

#### Element Operators
$exists, $type

```javascript
db.collection.find({"tomato.meter":{"$exists":true}})
```

#### Logical Operators
$or, $and 当查询条件中多次涉及同一字段时用$and，如 {$and:[{$grade:{$exists:true}}, {$grade:{$ne:null}}]}

```javascript
db.collection.find({$or: [{$field1:val1}, {field2:val2}] })
db.collection.find({$and: [{$field1:val1}, {field1:val2}] })
```

#### Regex Operator
$regex

```javascript
db.collection.find({"strfield":{$regex:"pattern"}})
```

#### Array Operators
$all, $size, $elementMatch

```javascript
db.collection.find({actors:{$all:["actor1", "actor2", "actor3"]}})
db.collection.find({actors:{$size:1}})
boxOffice: [ { "country": "USA", "revenue": 41.3 },
             { "country": "Australia", "revenue": 2.9 },
             { "country": "UK", "revenue": 10.1 },
             { "country": "Germany", "revenue": 4.3 },
             { "country": "France", "revenue": 3.5 } ]
db.collection.find({ boxOffice: { country: "UK", revenue: { $gt: 15 } } })
# 会匹配以上字段，country和revenue不被限制在同一元素内
db.collection.find({ boxOffice: {$elemMatch: { country: "UK", revenue: { $gt: 15 } } } })
# country和revenue被限制在同一元素内，匹配失败
```

## Update

### UpdateOne
[Update Operators](https://docs.mongodb.com/v3.2/reference/operator/update/): $inc, $mul, $rename, $setOnInsert, $set, $unset, $min, $max, $currentDate

```javascript
db.movieDetails.updateOne({title: "The Martian"},
                          { $set: { "awards" : {"wins" : 8,
		                              "nominations" : 14,
		                                "text" : "Nominated for 3 Golden Globes. Another 8 wins & 14 nominations." } } });
```

### Array Update
[Array Update Operators](https://docs.mongodb.com/v3.2/reference/operator/update-array/): $addToSet, $pop, $pull, $pullAll, $push, $pushAll

```javascript
db.movieDetails.updateOne({title: "The Martian"},
                          {$push: { reviews:
                                    { $each: [
                                        { rating: 0.5,
                                          date: ISODate("2016-01-12T07:00:00Z"),
                                          reviewer: "Yabo A.",
                                          text: "i believe its ranked high due to its slogan 'Bring him Home' there is nothing in the movie, nothing at all ! Story telling for fiction story !"},
                                        { rating: 5,
                                          date: ISODate("2016-01-12T09:00:00Z"),
                                          reviewer: "Kristina Z.",
                                          text: "This is a masterpiece. The ending is quite different from the book - the movie provides a resolution whilst a book doesn't."}],
				      $position: 0,
                                      $slice: 5 
				    }
				}
			})
```

$each 用于将列表中的文档作为单独的元素逐一插入，否则整个列表作为一个列表元素插入, $position:0 用于插入到列表前，$slice用于限定列表长度，详见文档

### UpdateMany
```javascript
db.movieDetails.updateMany( { rated: null },
                            { $unset: { rated: "" } } )
```

### Upsert
要更新的文档不存在时插入创建新的文档

```javascript
db.movieDetails.updateOne(
    {"imdb.id": detail.imdb.id},
    {$set: detail},
    {upsert: true});
```

### replaceOne
_id字段不变替换文档内容

```javascript
db.movies.replaceOne(
    {"imdb": detail.imdb.id},
    detail);
```

# Pymongo

## find_one, find, cursor, insert_one
find_one直接返回一个结果，find函数返回游标，游标可迭代。query与projection与mongo shell中语法一样，但python要求key加引号。

```python
import pymongo
connection = pymongo.MongoClient('mongodb://localhost')
db = connection.db_name
collection = db.collection_name
doc = collection.find_one(query, projection)
cursor = collection.find(query, projection)
collection.insert_one(new_doc)
```

## Sort, Skip and Limit
获取的查询结果总是按照sort, skip, limit的顺序进行，无论其在python中的调用顺序，因为只有在从cursor中读取结果集的时候才真正进行数据查询。  
pymongo的排序语法与mongo shell差别较大，因为mongo shell中的json保持了key的顺序，而python的字典则是无序的，所以采用列表。

```python
cursor.Skip(3)
cursor.limit(5)
cursor.sort([("field", pymongo.ASCENDING), ("field2", pymongo.DESCENDING)])
```

## insert_one
当未提供_id字段时插入，可能会出现重复数据。当提供_id字段时，如果数据库中已有_id值相同的文档则插入失败。

```python
richard = {"name": "Richard Kreuter", "company": "MongoDB",
               "interests": ['horses', 'skydiving', 'fencing']}
andrew = {"_id": "erlichson", "name": "Andrew Erlichson",
          "company": "MongoDB", "interests": ['running', 'cycling',
                                              'photography']}
people.insert_one(richard)
people.insert_one(andrew)
# Insert again
people.insert_one(richard) # succeed
people.insert_one(andrew)  # fail
```

## insert_many
ordered设为False时，某个文档插入失败不会影响后续文档插入。

```python
insert_many(doc_list, ordered=True)
```

## Update Data
与mongo shell语法相似。

```python
update_one(filter, update_statement, upsert=False)
update_many(filter, update_statement, upsert=False)
```

## replace_one
replace_one实际调用了update方法，在获取文档再替代的时间内原文档可能已被修改。

```python
replace_one(filter, new_doc)
```

## upsert
update和replace的upsert结果不同，下面例子中，update insert后插入的文档包含thing字段，而replace upsert后插入的文档不包含thing字段。

```python
update_one({"thing":"banana"},{"$set":{"color":"yellow"}},upsert=True)
update_many({"thing":"banana"},{"$set":{"color":"yellow"}},upsert=True)
replace_one({"thing":"banana"},{"$set":{"color":"yellow"}},upsert=True)
```

## delete_one, delete_many
没啥特殊的，就给个filter参数删除。。。

## find_and_modify
find_one_and_delete/replace/update()  
参数有 filter, update, projection, sort, upsert, return_document, **kwargs  
return_document 默认为BEFORE，即返回修改前的文档。

```python
db.test.find_one_and_update(
    {'done': True},
    {'$set': {'final': True}},
    sort=[('_id', pymongo.DESCENDING)])
db.example.find_one_and_update(
    {'_id': 'userid'},
    {'$inc': {'seq': 1}},
    projection={'seq': True, '_id': False},
    upsert=True,
    return_document=pymongo.ReturnDocument.AFTER)
```
