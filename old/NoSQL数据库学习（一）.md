title: NoSQL数据库学习（一）
date: 2016-01-10 11:28:30
tags: mongoDB 
categories: 大前端 
---

# 写在前面
还记得若干年前LAMP很火，Linux+Apache+MySQL+PHP这可谓是中小型网站建站的黄金之选。如今峰回路转，大前端时代nodeJS的到来，使得 MEAN（MongoDB+Express+Angular+NodeJS）成为另一个快速建站和大并发优化的不错之选。
# 什么是NoSQL
NoSQL，指的是非关系型的数据库。NoSQL有时也称作Not Only SQL的缩写，是对不同于传统的关系型数据库的数据库管理系统的统称。NoSQL用于超大规模数据的存储。（例如谷歌或Facebook每天为他们的用户收集万亿比特的数据）。这些类型的数据存储不需要固定的模式，无需多余操作就可以横向扩展。

<!--more-->

## RDBMS vs NoSQL
**RDBMS** 
> 
高度组织化结构化数据 
结构化查询语言（SQL） (SQL) 
数据和关系都存储在单独的表中。 
数据操纵语言，数据定义语言 
严格的一致性
基础事务

**NoSQL** 
> 
代表着不仅仅是SQL
没有声明性查询语言
没有预定义的模式
键 - 值对存储，列存储，文档存储，图形数据库
最终一致性，而非ACID属性
非结构化和不可预知的数据
CAP定理 
高性能，高可用性和可伸缩性

![这里写图片描述](http://img.blog.csdn.net/20160107103158595)

## CAP定理（CAP theorem）
在计算机科学中, CAP定理（CAP theorem）, 又被称作 布鲁尔定理（Brewer's theorem）, 它指出对于一个分布式计算系统来说，不可能同时满足以下三点：
> 
**一致性(Consistency)**  (所有节点在同一时间具有相同的数据)
**可用性(Availability)**  (保证每个请求不管成功或者失败都有响应)
**分隔容忍(Partition tolerance)**  (系统中任意信息的丢失或失败不会影响系统的继续运作)

CAP理论的核心是：一个分布式系统不可能同时很好的满足一致性，可用性和分区容错性这三个需求，最多只能同时较好的满足两个。因此，根据 CAP 原理将 NoSQL 数据库分成了满足 CA 原则、满足 CP 原则和满足 AP 原则三 大类：
> 
**CA** - 单点集群，满足一致性，可用性的系统，通常在可扩展性上不太强大。
**CP** - 满足一致性，分区容忍必的系统，通常性能不是特别高。
**AP** - 满足可用性，分区容忍性的系统，通常可能对一致性要求低一些。

![这里写图片描述](http://img.blog.csdn.net/20160107103607120)

## NoSQL的优点/缺点
**优点**
> 
高可扩展性
分布式计算
低成本
架构的灵活性，半结构化数据
没有复杂的关系

**缺点**
> 
没有标准化
有限的查询功能（到目前为止）
最终一致是不直观的程序

# 基本语法
## MongoDB 创建数据库
**语法：**`use DATABASE_NAME` （如果数据库不存在，则创建数据库，否则切换到指定数据库）
**实例：** 以下实例我们创建了数据库 `runoob`
```
> use runoob
switched to db runoob
> db
runoob
> 
```
如果你想查看所有数据库，可以使用 show dbs 命令：
```
> show dbsshow dbs
local  0.078GB
test   0.078GB
> 
```
可以看到，我们刚创建的数据库 runoob 并不在数据库的列表中， 要显示它，我们需要向 runoob 数据库插入一些数据。
```
> db.runoob.insert({"name":"菜鸟教程"})
WriteResult({ "nInserted" : 1 })
> show dbs
local   0.078GB
runoob  0.078GB
test    0.078GB
> 
```
MongoDB 中默认的数据库为 test，如果你没有创建新的数据库，集合将存放在 test 数据库中。

## MongoDB 删除数据库
**语法：** `db.dropDatabase()` （删除当前数据库，默认为 test，你可以使用 db 命令查看当前数据库名）

## MongoDB 插入文档
**语法：** MongoDB 使用 insert() 或 save() 方法向集合中插入文档，语法如下：

```
db.COLLECTION_NAME.insert(document)
```
## MongoDB 更新文档
MongoDB 使用 update() 和 save() 方法来更新集合中的文档。接下来让我们详细来看下两个函数的应用及其区别。
### **update() 方法**
update() 方法用于更新已存在的文档。语法格式如下：
```
db.collection.update(
   <query>,
   <update>,
   {
     upsert: <boolean>,
     multi: <boolean>,
     writeConcern: <document>
   }
)
```
> 参数说明：
**query :** update的查询条件，类似sql update查询内where后面的。
**update :** update的对象和一些更新的操作符（如$,$inc...）等，也可以理解为sql update查询内set后面的
**upsert :** 可选，这个参数的意思是，如果不存在update的记录，是否插入objNew,true为插入，默认是false，不插入。
**multi :** 可选，mongodb 默认是false,只更新找到的第一条记录，如果这个参数为true,就把按条件查出来多条记录全部更新。
**writeConcern :**可选，抛出异常的级别。

### **save() 方法**
save() 方法通过传入的文档来替换已有文档。语法格式如下：
```
db.collection.save(
   <document>,
   {
     writeConcern: <document>
   }
)
```
> 参数说明：
**document :** 文档数据。
**writeConcern :** 可选，抛出异常的级别。

## MongoDB 删除文档
**语法：** remove() 方法的基本语法格式如下所示：
```
db.collection.remove(
   <query>,
   {
     justOne: <boolean>,
     writeConcern: <document>
   }
)
```
> 参数说明：
**query :** （可选）删除的文档的条件。
**justOne :** （可选）如果设为 true 或 1，则只删除一个文档。
**writeConcern :**（可选）抛出异常的级别。

## MongoDB 查询文档
**语法：**  
```
//MongoDB 查询数据的语法格式如下
>db.COLLECTION_NAME.find();

//find() 方法以非结构化的方式来显示所有文档。
//如果你需要以易读的方式来读取数据，可以使用 pretty() 方法，语法格式如下：
>db.col.find().pretty();
```

**MongoDB 与 RDBMS Where 语句比较**

![这里写图片描述](http://img.blog.csdn.net/20160107143141397)

### MongoDB AND 条件
```
>db.col.find({key1:value1, key2:value2}).pretty()
```
### MongoDB OR 条件
```
db.col.find(
   {
      $or: [
	     {key1: value1}, {key2:value2}
      ]
   }
).pretty()
```
### AND 和 OR 联合使用
```
>db.col.find({"likes": {$gt:50}, $or: [{"by": "菜鸟教程"},{"title": "MongoDB 教程"}]}).pretty()
{
        "_id" : ObjectId("56063f17ade2f21f36b03133"),
        "title" : "MongoDB 教程",
        "description" : "MongoDB 是一个 Nosql 数据库",
        "by" : "菜鸟教程",
        "url" : "http://www.runoob.com",
        "tags" : [
                "mongodb",
                "database",
                "NoSQL"
        ],
        "likes" : 100
}
```

## MongoDB 条件操作符

条件操作符用于比较两个表达式并从mongoDB集合中获取数据。MongoDB中条件操作符有：

###  (>) 大于 - $gt 
```
db.col.find({"likes" : {$gt : 100}})
```
### (<) 小于 - $lt  
```
db.col.find({likes : {$gte : 100}})
```
### (>=) 大于等于 - $gte 
```
db.col.find({likes : {$lt : 150}})
```
### (<= ) 小于等于 - $lte
```
db.col.find({likes : {$lte : 150}})
```
### MongoDB 使用 (<) 和 (>) 查询 - $lt 和 $gt
```
db.col.find({likes : {$lt :200, $gt : 100}})
```

## MongoDB 排序
在MongoDB中使用使用sort()方法对数据进行排序，sort()方法可以通过参数指定排序的字段，并使用 1 和 -1 来指定排序的方式，其中 1 为升序排列，而-1是用于降序排列。
```
>db.COLLECTION_NAME.find().sort({KEY:1})
```
## MongoDB Limit() 方法
如果你需要在MongoDB中读取指定数量的数据记录，可以使用MongoDB的Limit方法，limit()方法接受一个数字参数，该参数指定从MongoDB中读取的记录条数。
```
>db.COLLECTION_NAME.find().limit(NUMBER)
```

## MongoDB Skip() 方法
我们除了可以使用limit()方法来读取指定数量的数据外，还可以使用skip()方法来跳过指定数量的数据，skip方法同样接受一个数字参数作为跳过的记录条数。
```
>db.COLLECTION_NAME.find().limit(NUMBER).skip(NUMBER)
```


> 参考 [《NoSQL 简介》](http://www.runoob.com/mongodb/nosql.html)


