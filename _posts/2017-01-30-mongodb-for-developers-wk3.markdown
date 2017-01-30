---
layout: 	post
title:		"Mongdb for Developers: Week3"
subtitle:	"Schema Design & Blob"
header-img:	"img/post-bg-deeplearning.jpg"
date:		2017-01-30
author: 	"Borg"
catalog:	true
tags:
    - Mongodb
---
# Schema Design
mongodb区别与sql数据库，没有外键，支持内嵌文档。在设计数据库时既可以内嵌也可以分别创建不同的集合再进行连接，较为自由。内嵌的好处一是提高磁盘读取效率。缺点是不能内嵌太多文档，否则会超过16MB的文档大小限制，还有如果内嵌导致大量重复数据则容易造成数据不一致。

## One to One
一对一关系直接使用内嵌。

## One to Many
一对多关系中，根据Many的数量多少可以进一步分出One to Few。如果是One to Few的情况可以使用内嵌。如果Many数量实在太大，则可以建立新的集合进行连接。

## Many to Many
多对多关系也可以分为Few to Few和Many to Many。可以建立单向或者双向连接，如果使用内嵌会导致数据重复（eg. 书和作者多对多关系中如果把书内嵌到作者中，一本书对应的文档会在多个读者文档中出现多次）。

## Tree
树结构的表示（不要使用链表那样的结构，即一个Category中只存上一级，因为需要寻找多次才能找到顶部）。  

Products:
  category: 7
  product_name: "toothbrush"

Categories:
  _id: 7
  category_name: "outdoors"
  ancestors: [3, 4 ,6]  # 按树的顺序从上到下排列

# Handling Blob
GridFS 用于把较大文件分割成多份存入Mongodb

```python
import pymongo
import gridfs

connection = pymongo.MongoClient()

db = connection.test
file_meta = db.file_meta
file_used = "sample_128_mb.txt"

grid = gridfs.GridFS(db, "text") # 第二个参数为集合名，默认为fs
fin = open(file_used, "rb")  # 下一行put方法不支持unicode，所以用python3的话需要用二进制方式读取

_id = grid.put(fin)
fin.close()
print "id of file is", _id

file_meta.insert( { "grid_id": _id, "filename": file_used})
```

# Assignment
## 3.1
数据集长这样：

```python
{
	"_id" : 3,
	"name" : "Bao Ziglar",
	"scores" : [
		{
			"type" : "exam",
			"score" : 71.64343899778332
		},
		{
			"type" : "quiz",
			"score" : 24.80221293650313
		},
		{
			"type" : "homework",
			"score" : 1.694720653897219
		},
		{
			"type" : "homework",
			"score" : 42.26147058804812
		}
	]
}
{
	"_id" : 0,
	"name" : "aimee Zank",
	"scores" : [
		{
			"type" : "exam",
			"score" : 1.463179736705023
		},
		{
			"type" : "quiz",
			"score" : 11.78273309957772
		},
		{
			"type" : "homework",
			"score" : 6.676176060654615
		},
		{
			"type" : "homework",
			"score" : 35.8740349954354
		}
	]
}
```
题目要求删除类型为**homework**的**最低**成绩，最后剩下三个不同类型的成绩。
偶的代码长这样：

```python
import pymongo

connection = pymongo.MongoClient()
db = connection.school
students = db.students

for ele in students.find({}):
    min_score = None
    for index, score in enumerate(ele['scores']):
        if score['type'] == "homework":
            if min_score is None:
                min_score = score['score']
            elif score['score'] < min_score:
                min_score = score['score']
    result = students.update({'_id':ele['_id']},{'$pull':{'scores':
                                {'type':"homework", 'score':min_score}}},multi=True)
```
十几行代码搞了好久，因为没搞清楚$elemenMatch的用法，误以为这里需要使用。最后是在[官方文档](https://docs.mongodb.com/manual/reference/operator/update/pull/#behavior)里找到了，原文如下：  

Because $pull operator applies its query to each element as though it were a 1top-level object, the expression did not require the use of $elemMatch to specify the condition of a score field equal to 8 and item field equal to "B".

所以其实是再内嵌一层文档才需要使用$elemMatch的。
