---
layout: post
title:  "MongoDB的基本操作总结"
date:   2016-02-17
author: 独上高楼
category: 编程
tags: [Python]
---

* [NoSql简介](http://www.runoob.com/mongodb/nosql.html)
* [MongoDB简介](http://www.runoob.com/mongodb/mongodb-intro.html)
* [在Windows平台下安装Mongo](http://www.runoob.com/mongodb/mongodb-window-install.html)
* [Mongo DB官方文档] (https://docs.mongodb.org/manual)

##MongoDB基本命令
* MongoDB的默认安装路径为 C:\Program Files\MongoDB
* 创建默认的数据库存放路径c:\data\db ,使用命令行把该路径关联到mongo上 
C:\Program Files\MongoDB\Server\3.2\bin\mongod.exe --dbpath c:\data\db，执行成功后数据库服务开启并开始监听

###Mongo Shell
* 运行 C:\Program Files\MongoDB\Server\3.2\bin\mongod.exe 文件可以打开MongoDB Shell，它是一个自带的交互式的JavaScript shell，用来对MongoDB进行操作和管理的交互式环境
* help 命令可以显示可使用的命令行

### DB相关的操作
 
    use tutorial

使用该命令会尝试连接名字为tutorial的数据库，如果不存在则创建。使用db.help()命令可以查看命令行帮助

    show dbs

显示数据库的相关信息.

如果数据库相关的名字里包含了空格等字符，也可以用下面的命令

    db["dbname"].find()
    db.getCollection("dbname").find()


### 插入数据

通过下面的格式来添加数据：

    db.restaurants.insert(
       {
      	"address" : {
     	"street" : "2 Avenue",
     	"zipcode" : "10075",
     	"building" : "1480",
     	"coord" : [ -73.9557413, 40.7720266 ],
      },
      "borough" : "Manhattan",
      "cuisine" : "Italian",
      "grades" : [
     {
    	"date" : ISODate("2014-10-01T00:00:00Z"),
    	"grade" : "A",
    	"score" : 11
     },
     {
    	"date" : ISODate("2014-01-16T00:00:00Z"),
    	"grade" : "B",
    	"score" : 17
     }
      ],
      "name" : "Vella",
      "restaurant_id" : "41704620"
       }
    )
    
### 查找数据

如果想要查找所有的数据，则：

    db.collectionname.find()

指定条件：

指定field条件进行筛选，使用如下格式：

    { <field1>: <value1>, <field2>: <value2>, ... }

具体的例子：

    db.restaurants.find( { "borough": "Manhattan" } )


大于，小于条件的筛选

    db.restaurants.find( { "grades.score": { $gt: 30 } } )
    db.restaurants.find( { "grades.score": { $lt: 10 } } )

AND和OR

    db.restaurants.find( { "cuisine": "Italian", "address.zipcode": "10075" } )
    db.restaurants.find( { $or: [ { "cuisine": "Italian" }, { "address.zipcode": "10075" } ] } )

排序

    db.restaurants.find().sort( { "borough": 1, "address.zipcode": 1 } )


### 更新数据

下面的操作更新name为Juni的记录，用$set 操作来更新cuisine 字段。 用 $currentDate 操作符来更新lastModified字段：

    db.restaurants.update(
    { "name" : "Juni" },
    {
      $set: { "cuisine": "American (New)" },
      $currentDate: { "lastModified": true }
    }
    )

更新内嵌的数据：

    db.restaurants.update(
      { "restaurant_id" : "41156888" },
      { $set: { "address.street": "East 31st Street" } }
    )

更新多条数据：
默认情况下update方法只更新一条数据。想要更新多条数据，使用multi option。

    db.restaurants.update(
      { "address.zipcode": "10016", cuisine: "Other" },
      {
	    $set: { cuisine: "Category To Be Determined" },
	    $currentDate: { "lastModified": true }
      },
      { multi: true}
    )


替换某条记录
根据某个_id字段的信息，用新的记录替换就得

    db.restaurants.update(
       { "restaurant_id" : "41704620" },
       {
	     "name" : "Vella 2",
	     "address" : {
	      "coord" : [ -73.9557413, 40.7720266 ],
	      "building" : "1480",
	      "street" : "2 Avenue",
	      "zipcode" : "10075"
       	  }
       }
    )

### 删除某条记录

删除符合某个条件的所有记录：

    db.restaurants.remove( { "borough": "Manhattan" } )

只删除符合某个条件的一条记录，使用justOne选项：

    db.restaurants.remove( { "borough": "Queens" }, { justOne: true } )

删除所有的记录：

    db.restaurants.remove( { } )

删除一个表：

    db.restaurants.drop()

### 聚合运算
###聚合并累加

用$group 来通过某个关键字进行分组，在$group中，指定需要分组的关键字为_id。$group通过field path访问字段，字段名字需要以$为前缀。$sum表示累加器，下面的语句表示计算字段为borough的各种情况的个数。

    db.restaurants.aggregate(
       [
     	{ $group: { "_id": "$borough", "count": { $sum: 1 } } }
       ]
    );

输出结果为：

    { "_id" : "Staten Island", "count" : 969 }
    { "_id" : "Brooklyn", "count" : 6086 }
    { "_id" : "Manhattan", "count" : 10259 }
    { "_id" : "Queens", "count" : 5656 }
    { "_id" : "Bronx", "count" : 2338 }
    { "_id" : "Missing", "count" : 51 }


#### 聚合并过滤

使用 $match 来过滤记录

    db.restaurants.aggregate(
       [
	     { $match: { "borough": "Queens", "cuisine": "Brazilian" } },
	     { $group: { "_id": "$address.zipcode" , "count": { $sum: 1 } } }
       ]
    );

