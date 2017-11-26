title: "MongoDB"
date: 2016-03-22 20:55:47
author: mzeht
avatar: /images/favicon.png
categories: 
 - 数据库
 - MongoDb
tags: 
- MongoDb
---

#简单介绍
##简介
MongoDB是一个基于分布式文件存储的数据库，由C++语言编写，旨在为WEB应用提供可扩展的高性能数据存储解决方案。

MongoDB是一个高性能，开源，无模式的文档型数据库，官方给自己的定义是Key-value存储(高性能和高扩展)和传统RDBMS(丰富的查询和功能)之间的一座桥梁


<!-- more -->
##Document和BSON
MongoDB中保存的数据格式为BSON，如：。
![](http://7xqtsx.com1.z0.glb.clouddn.com/bson.jpg)


MongoDB中数据的基本单元称为文档(Document)，它是MongoDB的核心概念，由多个键极其关联的值有序的放置在一起组成，数据库中它对应于关系型数据库的行。

数据在MongoDB中以BSON（Binary-JSON）文档的格式存储在磁盘上。

BSON（Binary Serialized Document Format）是一种类json的一种二进制形式的存储格式，简称Binary JSON，BSON和JSON一样，支持内嵌的文档对象和数组对象，但是BSON有JSON没有的一些数据类型，如Date和BinData类型。

BSON的优点是灵活性高，但它的缺点是空间利用率不是很理想，BSON有三个特点：轻量性、可遍历性、高效性。


#文档的增删查改
##简单插入文档
在数据库中，数据插入是最基本的操作，在MongoDB使用db.collection.insert(document)语句来插入文档，如下图：
![](http://7xqtsx.com1.z0.glb.clouddn.com/insert1.png)

document是文档数据，collection是存放文档数据的集合。

例如：所有用户的信息存放在users集合中，每个用户的信息为一个user文档，插入数据:

db.users.insert(user);
如果collection存在，document会添加到collection目录下， 如果collection不存在，数据库会先创建collection，然后再保存document。

练习一下：把自己的姓名(name)和年龄(age)保存到person集合中！
如果想要查看已插入的person文档，可以使用：db.person.find()查看当前库中person集合里的数据。

如果想要查看当前数据库中的集合列表，可以使用：show collections。

db.person.insert({name:'阿牛', age:28});

##批量插入文档
insert语句不但可以插入单个文档，还可以一次性插入多个文档。

插入多个文档时，insert命令的参数为一个数组，数组元素为BSON格式的文档。

多个文档可以放在一个数组内，一次插入多条数据，例如：

```
db.users.insert([{name:"tommy"},{name:"xiaoming"}])
```
文档批量插入非常方便，但是使用批量插入时也有一些问题需要注意，因为BSON格式的限制，一次插入的数据量不能超过16M，在一个insert命令中插入多条数据时，MongoDB不保证完全成功或完全失败。

使用insert命令将以下数据一次性插入person集合中。


文／Te_Lee（简书作者）
原文链接：http://www.jianshu.com/p/1e402922ee32/
著作权归作者所有，转载请联系作者获得授权，并标注“简书作者”。

|name|age   |status|
|----|:----:|:-----:|
|mary|21    |A     |
|Lucy|89    |A     |
| Jacky |30    |A     |

```
db.person.insert([
 {name:"Mary",  age:21, status:"A"},
 {name:"Lucy",  age:89, status:"A"},
 {name:"Jacky", age:30, status:"A"} 
]);
```


##查询文档
在MongoDB中，查询指向特定的文档集合，查询设定条件，指明MongoDB需要返回的文档；查询也可以包含一个投影，指定返回的字段。

如下图，在查询过程指定了一个查询条件和一个排序修饰。



