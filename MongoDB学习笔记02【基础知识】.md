在MongoDB中基本的概念是文档、集合、数据库，下图与RDBMS对比，比较容易理解。

SQL术语|MongoDB术语|解释
---|---|---
database|	database|	数据库
table|	collection	|数据库表/集合
row|	document	|数据记录行/文档
column|	field	|数据字段/域
index|	index	|索引
table |joins	 |	表连接,MongoDB不支持
primary key|	primary key	|主键,MongoDB自动将_id字段设置为主键

### 数据库DataBase
一个MongoDB可以建多个数据库。MongoDB的默认数据库为"db",即启动时前创建的"/data/db"目录。
#### 常用命令
* 查看数据库 `show dbs`
	> 如果数据库为空，则不会显示
* 显示当前数据库对象 `db`
* 切换/创建数据库 `use local`
* 删除数据库 `db.dropDatabase()`

<!-- more -->	
#### 命名规范
数据库的命名需符合以下条件：
> 1 不能是空字符串（"")。
> 2 不得含有' '（空格)、.、$、/、\和\0 (空宇符)。
> 3 应全部小写。
> 4 最多64字节。

注意有一些数据库名是保留的：
> **admin：**从权限的角度来看，这是"root"数据库。要是将一个用户添加到这个数据库，这个用户自动继承所有数据库的权限。一些特定的服务器端命令也只能从这个数据库运行，比如列出所有的数据库或者关闭服务器。
> **local：** 这个数据永远不会被复制，可以用来存储限于本地单台服务器的任意集合
> **config：** 当Mongo用于分片设置时，config数据库在内部使用，用于保存分片的相关信息。

### 文档Document
文档是一个键值(key-value)对(即BSON)。MongoDB 的文档不需要设置相同的字段，并且相同的字段不需要相同的数据类型。例如：
```
{"site":"bochenlong.github.io", "name":"MongoDB学习笔记本02"}
```
需要注意的是：
* 文档中的键/值对是有序的。
* 文档中的值不仅可以是在双引号里面的字符串，还可以是其他几种数据类型（甚至可以是整个嵌入的文档)。
* MongoDB区分类型和大小写。
* MongoDB的文档不能有重复的键。
* 文档的键是字符串。除了少数例外情况，键可以使用任意UTF-8字符。

#### 常用命令
##### 插入文档 
`db.COLLECTION_NAME.insert(document)`
  > 如果集合不存在，会自动创建
  > 亦可以定义document=({})变量再做插入

##### 查看文档 
`db.COLLECTION_NAME.find()`
输出结果美观 `db.COLLECTION_NAME.find().pretty()`
查看一个文档 `db.COLLECTION_NAME.findOne()`
查看指定个数文档 `db.COLLECTION_NAME.find().limit(NUMBER)`
跳过指定个数文档 `db.COLLECTION_NAME.find().skip(NUMBER)`
指定排序查看文档 `db.CONLLECTION_NAME.find().sort({key:value}}`
> 1 和 -1 来指定排序的方式，其中 1 为升序排列，而-1是用于降序排列。默认按照文档的升序排列

##### 更新文档 
```
db.COLLECTION_NAME.update(
    <query>,
    <update>,
    {
      upsert:<boolean>,
      multi:<boolean>,
      writeConcern:<document>
    }
)
```
参数说明：
> **query :** update的查询条件，类似sql update查询内where后面的。
> **update :** update的对象和一些更新的操作符（如$,$inc...）等，也可以理解为sql update查询内set后面的
> **upsert :** 可选，这个参数的意思是，如果不存在update的记录，是否插入objNew,true为插入，默认是false，不插入。
> **multi :** 可选，mongodb 默认是false,只更新找到的第一条记录，如果这个参数为true,就把按条件查出来多条记录全部更新。
> **writeConcern :**可选，抛出异常的级别。

 例如：`db.user.update({'name':'bochenlong'},{$set:{'name':'bochenlong1'}})`

##### 替换文档 
```
db.COLLECTION_NAME.save(
   <document>,
   {
     writeConcern: <document>
   }
)
```
参数说明：
> **document :** 文档数据
> **writeConcern :**可选，抛出异常的级别。

 例如：
```
db.user.save({"_id" : ObjectId("5762436b7b4000ad140079ef"), "name" : "bochenlong2", "age" : 1})
```
##### 删除文档
```
db.COLLECTION_NAME.remove(
<query>,
  {
    justOne: <boolean>,
    writeConcern: <document>
  }
)
```
参数说明：
> **query :**（可选）删除的文档的条件。
> **justOne : **（可选）如果设为 true 或 1，则只删除一个文档。
> **writeConcern :**（可选）抛出异常的级别。

 例如：`db.user.remove({"name":"bochenlong2"},{justOne:true})`

#### 命名规范
文档键命名规范：
* 键不能含有\0 (空字符)。这个字符用来表示键的结尾。
* .和$有特别的意义，只有在特定环境下才能使用。
* 以下划线"_"开头的键是保留的(不是严格要求的)。


#### 条件操作符

操作|格式|范例
---|---|---
等于|`{<key>:<value>}`|db.user.find({"name":"bochenlong"}).pretty()
小于|`{<key>:{$lt:<value>}}`	|db.user.find({"age":{$lt:50}}).pretty()
小于或等于|	`{<key>:{$lte:<value>}}`	|db.user.find({"age":{$lte:50}}).pretty()	
大于|	`{<key>:{$gt:<value>}}`|	db.user.find({"age":{$gt:50}}).pretty()	
大于或等于|	`{<key>:{$gte:<value>}}`	|db.user.find({"age":{$gte:50}}).pretty()
不等于|	`{<key>:{$ne:<value>}}`|	db.user.find({"age":{$ne:50}}).pretty()
AND | `{key1:value1, key2:value2...}`|db.user.find({"age":{$lt:50},"name":"bochenlong"})
OR | `{$or:[key1:value1, key2:value2...]}`|db.user.find({$or:[{"age":{$lt:50}},{"name":"bochenlong"}]})
$type | `{<key>:{$type:<value>}}`|db.user.find({"name":{$type:2}})

> $type的value主要为下： Double-1,String-2,Object-3,Array-4,Binary data-5,Undefined-6(已废弃),Object id-7,Boolean-8,Date-9,Null-10,Regular Expression-11,JavaScript-13,Symbol-14,JavaScript (with scope)-15,32-bit integer-16,Timestamp-17,64-bit integer-18,Min key-225(Query with -1),Max key-127

### 集合Collection
集合就是 MongoDB 文档组。集合存在于数据库中，集合没有固定的结构，这意味着你在对集合可以插入不同格式和类型的数据，但通常情况下我们插入集合的数据都会有一定的关联性。比如，我们可以将以下不同数据结构的文档插入到集合中：
```
{"site":"bochenlong.github.io", "name":"MongoDB学习笔记本02"}
{"site":"bochenlong.github.io", "name":"MongoDB学习笔记本02",num:"001"}
```
当第一个文档插入时，集合就会被创建。

#### 命名规范
* 集合名不能是空字符串""。
* 集合名不能含有\0字符（空字符)，这个字符表示集合名的结尾。
* 集合名不能以"system."开头，这是为系统集合保留的前缀。
* 用户创建的集合名字不能含有保留字符。有些驱动程序的确支持在集合名里面包含，这是因为某些系统生成的集合中包含该字符。除非你要访问这种系统创建的集合，否则千万不要在名字里出现$。　

#### Capped Collection
Capped Collection就是固定大小的Collection。它有很**高的性能以及队列过期的特性**(过期按照插入的顺序)。
Capped Collections是高性能自动的维护对象的插入顺序。它非常适合类似记录日志的功能。和标准的collection不同，你必须要显式的创建一个Capped Collection,指定一个collection的大小，单位是字节。Collection的数据存储空间值是提前分配的。
* 创建 `db.createCollection("mycc", {capped:true, size:1000})`

注意：
> 在capped collection中，你能添加新的对象。
> 能进行更新，然而，对象不会增加存储空间。如果增加，更新就会失败 。
> 数据库不允许进行删除。使用drop()方法删除collection所有的行。
> 删除之后，你必须显式的重新创建这个collection。
> 在32bit机器中，capped collection最大存储为1e9个字节。

### 元数据
数据库的信息是存储在集合中。它们使用了系统的命名空间 `dbname.system.*`；在MongoDB数据库中名字空间 <dbname>.system.* 是包含多种系统信息的特殊集合(Collection)，如下:

集合命名空间|描述
---|---
dbname.system.namespaces|	列出所有名字空间。
dbname.system.indexes|	列出所有索引。
dbname.system.profile|	包含数据库概要(profile)信息。
dbname.system.users|	列出所有可访问数据库的用户。
dbname.local.sources|	包含复制对端（slave）的服务器信息和状态。
在`system.indexes`插入数据，可以创建索引。但除此之外该表信息是不可变的(特殊的drop index命令将自动更新相关信息)。`system.users`是可修改的。`system.profile`是可删除的。

### 数据类型

数据类型|描述
---|---
String|	字符串。存储数据常用的数据类型。在 MongoDB 中，UTF-8 编码的字符串才是合法的。
Integer|	整型数值。用于存储数值。根据你所采用的服务器，可分为 32 位或 64 位。
Boolean|	布尔值。用于存储布尔值（真/假）。
Double|	双精度浮点值。用于存储浮点值。
Min/Max keys|	将一个值与 BSON（二进制的 JSON）元素的最低值和最高值相对比。
Arrays|	用于将数组或列表或多个值存储为一个键。
Timestam|p	时间戳。记录文档修改或添加的具体时间。
Object|	用于内嵌文档。
Null|	用于创建空值。
Symbol|	符号。该数据类型基本上等同于字符串类型，但不同的是，它一般用于采用特殊符号类型的语言。
Date|	日期时间。用 UNIX 时间格式来存储当前日期或时间。你可以指定自己的日期时间：创建 Date 对象，传入年月日信息。
Object ID|	对象 ID。用于创建文档的 ID。
Binary Data|	二进制数据。用于存储二进制数据。
Code|	代码类型。用于在文档中存储 JavaScript 代码。
Regular expression|	正则表达式类型。用于存储正则表达式。