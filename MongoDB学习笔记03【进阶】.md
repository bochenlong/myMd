### MongoDB索引
索引通常能够极大的提高查询的效率，如果没有索引，MongoDB在读取数据时必须扫描集合中的每个文件并选取那些符合查询条件的记录。索引是特殊的数据结构，索引存储在一个易于遍历读取的数据集合中，索引是对数据库表中一列或多列的值进行排序的一种结构。

#### 语法
`db.COLLECTION_NAME.ensureIndex({KEY:1,...})`
> 语法中KEY值为你要创建的索引字段，1为指定按升序创建索引，-1为按降序。

ensureIndex()接收可选参数，参见列表：
<!-- more -->

Parameter | Type | Description
---|---|---
background|boolean|	建索引过程会阻塞其它数据库操作，background可指定以后台方式创建索引，即增加 "background" 可选参数。 "background" 默认值为false
unique|boolean|建立的索引是否唯一。指定为true创建唯一索引。默认值为false
name|string|索引的名称。如果未指定，MongoDB的通过连接索引的字段名和排序顺序生成一个索引名称
dropDups|boolean|在建立唯一索引时是否删除重复记录,指定 true 创建唯一索引。默认值为 false
sparse|boolean|对文档中不存在的字段数据不启用索引；这个参数需要特别注意，如果设置为true的话，在索引字段中不会查询出不包含对应字段的文档.。默认值为 false
expireAfterSeconds|integer|指定一个以秒为单位的数值，完成 TTL设定，设定集合的生存时间
v|index version|索引的版本号。默认的索引版本取决于mongod创建索引时运行的版本
weights|	document|	索引权重值，数值在 1 到 99,999 之间，表示该索引相对于其他索引字段的得分权重。
default_language|	string	|对于文本索引，该参数决定了停用词及词干和词器的规则的列表。 默认为英语
language_override|	string|	对于文本索引，该参数指定了包含在文档中的字段名，语言覆盖默认的language，默认值为 language.

#### 实例
`db.user.ensureIndex({name:1},{background:true})`

### MongoDB聚合
MongoDB中聚合(aggregate)主要用于处理数据(诸如统计平均值,求和等)，并返回计算后的数据结果。

#### 语法
`db.COLLECTION_NAME.aggregate(AGGREGATE_OPERATION)`

#### 管道操作符

管道是由一个个功能节点组成的，这些节点用管道操作符来进行表示。聚合管道以一个集合中的所有文档作为开始，然后这些文档从一个操作节点流向下一个节点 ，每个操作节点对文档做相应的操作。这些操作可能会创建新的文档或者过滤掉一些不符合条件的文档，在管道中可以对文档进行重复操作。

**以下为某班成绩表：**
> 
 ```
{ "_id" : ObjectId("5767a91ef4de94710e10420e"), "name" : "bochenlong", "team" : "1", "yuwen" : 121, "shuxue" : 118, "yingyu" : 137, "lizong" : [ 90, 72, 66 ] }
{ "_id" : ObjectId("5767a936f4de94710e10420f"), "name" : "jiaoxinkang", "team" : "2", "yuwen" : 110, "shuxue" : 128, "yingyu" : 137, "lizong" : [ 80, 92, 77 ] }
{ "_id" : ObjectId("5767a959f4de94710e104210"), "name" : "lisan", "team" : "3", "yuwen" : 100, "shuxue" : 108, "yingyu" : 127, "lizong" : [ 70, 62, 88 ] }
{ "_id" : ObjectId("5767a96df4de94710e104211"), "name" : "lisan", "team" : "3", "yuwen" : 110, "shuxue" : 112, "yingyu" : 107, "lizong" : [ 90, 92, 68 ] }
{ "_id" : ObjectId("5767a97df4de94710e104212"), "name" : "lisi", "team" : "3", "yuwen" : 110, "shuxue" : 112, "yingyu" : 107, "lizong" : [ 90, 92, 68 ] }
{ "_id" : ObjectId("5767a9d4f4de94710e104213"), "name" : "mingren", "team" : "2", "yuwen" : 98, "shuxue" : 62, "yingyu" : 55, "lizong" : [ 20, 32, 58 ] }
 ```

##### $project
**数据投影，主要用于重命名、增加和删除字段（默认情况下_id是存在，如果不想显示手动置为0)**
> 示例：输出名称和语文成绩,且语文key声明为yw
> `db.score.aggregate({$project:{_id:0,name:1,yw:'$yuwen'}})`
> 输出结果：
```
{ "name" : "bochenlong", "yw" : 121 }
{ "name" : "jiaoxinkang", "yw" : 110 }
{ "name" : "lisan", "yw" : 100 }
{ "name" : "lisan", "yw" : 110 }
{ "name" : "lisi", "yw" : 110 }
{ "name" : "mingren", "yw" : 98 }
```

##### $match
**过滤操作，筛选符合条件文档，作为下一阶段的输入**
> 示例：输出名称和语文成绩，要求语文成绩大于100
> `db.score.aggregate([{$match:{'yuwen':{$gt:100}}},{$project:{'_id':0,'name':1,'yw':'$yuwen'}}])`
> 输出结果：
```
{ "name" : "bochenlong", "yw" : 121 }
{ "name" : "jiaoxinkang", "yw" : 110 }
{ "name" : "lisan", "yw" : 110 }
{ "name" : "lisi", "yw" : 110 }
```

##### $limit <span id='limit'></span>
**限制经过管道的文档数量**
> 示例：获取语文成绩前两名的名字和成绩
> `db.score.aggregate({$sort:{'yuwen':-1}},{$limit:2},{$project:{'_id':0,'name':1,'yw':'$yuwen'}})`
> 输出结果：
```
{ "name" : "bochenlong", "yw" : 121 }
{ "name" : "jiaoxinkang", "yw" : 110 }
```

##### $skip
**从代操作集合开始的位置跳过文档的数目**
> 示例：获取语文成绩第三名第四名的名字和成绩
> `db.score.aggregate({$sort:{'yuwen':-1}},{$skip:2},{$limit:2},{$project:{'_id':0,'name':1,'yw':'$yuwen'}})`
> 输出结果：
```
{ "name" : "jiaoxinkang", "yw" : 110 }
{ "name" : "lisan", "yw" : 110 }
```

##### $unwind
**将数组元素拆分为独立字段**
> 示例：获取name:bochenlong的理综各科成绩
> `db.score.aggregate({$project:{'_id':0,'name':1,'lizong':1}},{$match:{'name':'bochenlong'}},{$unwind:'$lizong'})`
> 输出结果：
```
{ "name" : "bochenlong", "lizong" : 90 }
{ "name" : "bochenlong", "lizong" : 72 }
{ "name" : "bochenlong", "lizong" : 66 }
```

##### $group <span id='group'></span>
**对数据进行分组**
> 示例：计算总人数
> `db.score.aggregate({$group:{'_id':null,num:{$sum:1}}})`
> 输出结果：
```
{ "_id" : null, "num" : 6 }
```

##### $sort
**对文案按照指定字段排序**
[参见$limit](#limit)

##### $goNear
**返回一些坐标值，这些指以按照举例指定点距离近到远进行排序**
[参见官网](https://docs.mongodb.com/manual/reference/operator/aggregation/geoNear/#pipe._S_geoNear)

#### 聚合表达式

管道操作符作为“键”,所对应的“值”叫做管道表达式。每个管道表达式只能作用于处理当前正在处理的文档，而不能进行跨文档的操作。管道表达式对文档的处理都是在内存中进行的。除了能够进行累加计算的管道表达式外，其他的表达式都是无状态的，也就是不会保留上下文的信息。
**组聚合操作符：**

##### $sum
**计算总和**
[参见$group](#group)

##### $avg
**计算平均值**
> 示例：计算语文平均成绩
> `db.score.aggregate({$group:{'_id':null,'yuwen':{$avg:'$yuwen'}}},{$project:{_id:0,'yuwen':1}})`
> 输出结果：
```
{ "yuwen" : 108.16666666666667 }
```

##### $min
**获取集合中所有文档对应值得最小值**
> 示例：计算语文最低成绩
> `db.score.aggregate({$group:{'_id':null,'yuwen':{$min:'$yuwen'}}},{$project:{_id:0,'yuwen':1}})`
> 输出结果：
```
{ "yuwen" : 98 }
```

##### $max
**获取集合中所有文档对应值得最大值**
> 示例：计算语文最高成绩
> `db.score.aggregate({$group:{'_id':null,'yuwen':{$max:'$yuwen'}}},{$project:{_id:0,'yuwen':1}})`
> 输出结果：
```
{ "yuwen" : 121 }
```

##### $push
**在结果文档中插入值到一个数组中**
> 示例：获取所有的语文成绩
> `db.score.aggregate({$group:{'_id':null,'yuwen':{$push:'$yuwen'}}},{$project:{_id:0,'yuwen':1}})`
> 输出结果：
```
{ "yuwen" : [ 121, 110, 100, 110, 110, 98 ] }
```

##### $addToSet
**在结果文档中插入值到一个数组中，如果数组中已有，则不插入**
> 示例：获取所有的语文成绩
> `db.score.aggregate({$group:{'_id':null,'yuwen':{$addToSet:'$yuwen'}}},{$project:{_id:0,'yuwen':1}})`
> 输出结果：
```
{ "yuwen" : [ 98, 100, 110, 121 ] }
```

##### $first
**根据资源文档的排序获取第一个文档数据**
> 示例：获取文档第一个数据
> `db.score.aggregate({$group:{'_id':null,'yuwen':{$first:'$yuwen'}}})`
> 输出结果：
```
{ "_id" : null, "yuwen" : 121 }
```

##### $last
**根据资源文档的排序获取最后一个文档数据**
> 示例：获取文档最后数据
> `db.score.aggregate({$group:{'_id':null,'yuwen':{$last:'$yuwen'}}})`
> 输出结果：
```
{ "_id" : null, "yuwen" : 98 }
```

#### Bool类型聚合操作符
##### $and
**根据条件输出true 或者 false**
> 示例：查看学生是否数学和语文都过100分
> `db.score.aggregate({$project:{'_id':0,'name':1,'result':{$and:[{$gt:['$yuwen',100]},{$gt:['$shuxue',100]}]}}})`
> 输出结果：
```
{ "name" : "bochenlong", "result" : true }
{ "name" : "jiaoxinkang", "result" : true }
{ "name" : "lisan", "result" : false }
{ "name" : "lisan", "result" : true }
{ "name" : "lisi", "result" : true }
{ "name" : "mingren", "result" : false }
```

##### $or
**根据条件输出true 或者 false**
> 示例：查看学生是否数学或语文过100分
> `db.score.aggregate({$project:{'_id':0,'name':1,'result':{$or:[{$gt:['$yuwen',100]},{$gt:['$shuxue',100]}]}}})`
> 输出结果：
```
{ "name" : "bochenlong", "result" : true }
{ "name" : "jiaoxinkang", "result" : true }
{ "name" : "lisan", "result" : true }
{ "name" : "lisan", "result" : true }
{ "name" : "lisi", "result" : true }
{ "name" : "mingren", "result" : false }
```

##### $not
**根据条件输出true 或者 false**
> 示例：查看学生是否语文不超过100分
> `db.score.aggregate({$project:{'_id':0,'name':1,'result':{$not:[{$gt:['$yuwen',100]}]}}}`
> 输出结果：
```
{ "name" : "bochenlong", "result" : true }
{ "name" : "jiaoxinkang", "result" : true }
{ "name" : "lisan", "result" : true }
{ "name" : "lisan", "result" : true }
{ "name" : "lisi", "result" : true }
{ "name" : "mingren", "result" : false }
```

#### 比较类型聚合操作符

##### $cmp
**比较两个值，返回数值类型**
> 示例：查看学生是否语文超过100分
> `db.score.aggregate({$project:{'_id':0,'name':1,'result':{$cmp:['$yuwen',100]}}})`
> 输出结果：
```
{ "name" : "bochenlong", "result" : 1 }
{ "name" : "jiaoxinkang", "result" : 1 }
{ "name" : "lisan", "result" : 0 }
{ "name" : "lisan", "result" : 1 }
{ "name" : "lisi", "result" : 1 }
{ "name" : "mingren", "result" : -1 }
```

##### $eq <span id='eq'></span>
**比较两个值是否相等，返回true或false**
> 示例：查看学生是否语文是100分（好无聊的要求）
> `db.score.aggregate({$project:{'_id':0,'name':1,'result':{$eq:['$yuwen',100]}}})`
> 输出结果：
```
{ "name" : "bochenlong", "result" : false }
{ "name" : "jiaoxinkang", "result" : false }
{ "name" : "lisan", "result" : true }
{ "name" : "lisan", "result" : false }
{ "name" : "lisi", "result" : false }
{ "name" : "mingren", "result" : false }
```

##### $gt
**比较是否大于，返回true或false**
[参见$eq](#eq)
##### $gte
**比较是否大于等于，返回true或false**
[参见$eq](#eq)
##### $lt
**比较是否小于，返回true或false**
[参见$eq](#eq)
##### $lte
**比较是否小于等于，返回true或false**
[参见$eq](#eq)
##### $ne
**比较是否不等于，返回true或false**
[参见$eq](#eq)

#### 算数类型聚合操作符
##### $add <span id='add'></span>
**计算数组的和**
> 示例：查看学生语数外成绩总和
> `db.score.aggregate({$project:{'_id':0,'name':1,'total':{$add:['$yuwen','$shuxue','$yingyu']}}})`
> 输出结果：
```
{ "name" : "bochenlong", "total" : 376 }
{ "name" : "jiaoxinkang", "total" : 375 }
{ "name" : "lisan", "total" : 335 }
{ "name" : "lisan", "total" : 329 }
{ "name" : "lisi", "total" : 329 }
{ "name" : "mingren", "total" : 215 }
```

##### $divide
**计算两数相除**
> 示例：查看学生语文成绩，按100分制
> `db.score.aggregate({$project:{'_id':0,'name':1,'yuwen':{$divide:['$yuwen',1.5]}}})`
> 输出结果：
```
{ "name" : "bochenlong", "yuwen" : 80.66666666666667 }
{ "name" : "jiaoxinkang", "yuwen" : 73.33333333333333 }
{ "name" : "lisan", "yuwen" : 66.66666666666667 }
{ "name" : "lisan", "yuwen" : 73.33333333333333 }
{ "name" : "lisi", "yuwen" : 73.33333333333333 }
{ "name" : "mingren", "yuwen" : 65.33333333333333 }
```

##### $mod
**计算两数取摩**
[参见$add](#add)

##### $multiply
**计算两数相乘**
[参见$add](#add)

##### $subtract
**计算两数相减**
[参见$add](#add)

#### 字符串类型聚合操作符
##### $concat
**字符串追加**
> 示例：讲学生和小组串联成一个字段
> `db.score.aggregate({$project:{'_id':0,'name-team':{$concat:['$name','-','$team']}}})`
> 输出结果：
```
{ "name-team" : "bochenlong-1" }
{ "name-team" : "jiaoxinkang-2" }
{ "name-team" : "lisan-3" }
{ "name-team" : "lisan-3" }
{ "name-team" : "lisi-3" }
{ "name-team" : "mingren-2" }
```

##### $strcasecmp
**忽略大小写比较字符串**
> 示例：将学生和小组串联成一个字段
> `db.score.aggregate({$project:{'_id':0,'name-team':{$concat:['$name','-','$team']}}})`
> 输出结果：
```
{ "name-team" : "bochenlong-1" }
{ "name-team" : "jiaoxinkang-2" }
{ "name-team" : "lisan-3" }
{ "name-team" : "lisan-3" }
{ "name-team" : "lisi-3" }
{ "name-team" : "mingren-2" }
```

##### $substr
**取子串**
> 示例：
> `db.score.aggregate({$project:{'_id':0,'name':{$substr:['$name',0,1]}}})`
> 输出结果：
```
{ "name" : "b" }
{ "name" : "j" }
{ "name" : "l" }
{ "name" : "l" }
{ "name" : "l" }
{ "name" : "m" }
```

##### $toUpper <span id='toUpper'></span>
**将字符串转换为大写**
> 示例：
> `db.score.aggregate({$project:{'_id':0,'name':{$toUpper:'$name'}}})`
> 输出结果：
```
{ "name" : "BOCHENLONG" }
{ "name" : "JIAOXINKANG" }
{ "name" : "LISAN" }
{ "name" : "LISAN" }
{ "name" : "LISI" }
{ "name" : "MINGREN" }
```

##### $toLower
**将字符串转换为小写**
[参见$toUpper](#toUpper)

#### 日期类型聚合操作符
Name|Description
---|---
$dayOfYear|获取天（1-366）
$dayOfMonth|获取月（1-31）
$dayOfWeek|获取星期（1-7）
$year|获取年
$month|获取月（1-12）
$week|获取星期（0-53）
$hour|获取分（0-23）
$minute|获取分（0-59）
$second|获取秒（0-59）
$millisecond|获取毫秒（0-999）

> 示例文档：
{ "_id" : 1, "item" : "abc", "price" : 10, "quantity" : 2, "date" : ISODate("2014-01-01T08:15:39.736Z") }
```
db.sales.aggregate(
   [
     {
       $project:
         {
           year: { $year: "$date" },
           month: { $month: "$date" },
           day: { $dayOfMonth: "$date" },
           hour: { $hour: "$date" },
           minutes: { $minute: "$date" },
           seconds: { $second: "$date" },
           milliseconds: { $millisecond: "$date" },
           dayOfYear: { $dayOfYear: "$date" },
           dayOfWeek: { $dayOfWeek: "$date" },
           week: { $week: "$date" }
         }
     }
   ]
)
```
输出结果：
```
{
  "_id" : 1,
  "year" : 2014,
  "month" : 1,
  "day" : 1,
  "hour" : 8,
  "minutes" : 15,
  "seconds" : 39,
  "milliseconds" : 736,
  "dayOfYear" : 1,
  "dayOfWeek" : 4,
  "week" : 0
}
```

#### 条件类型聚合操作符

##### $cond <span id='cond'></span>
**语法**
`{ $cond: { if: <boolean-expression>, then: <true-case>, else: <false-case-> } }`
or
`{ $cond: [ <boolean-expression>, <true-case>, <false-case> ] }`
> 示例：筛选出全班总分600以上含600分为优秀，500分含500分良，其它差
> `db.score.aggregate({$unwind:'$lizong'},{$group:{'_id':'$_id','name':{$first:'$name'},'yuwen':{$first:'$yuwen'},'shuxue':{$first:'$shuxue'},'yingyu':{$first:'$yingyu'},lizong:{$sum:'$lizong'}}},{$project:{'name':1,'zf':{$add:['$yuwen','$shuxue','$yingyu','$lizong']}}},{$project:{'name':1,'zf':1,'flag':{$cond: [ {$gte:['$zf',600]},'优秀',{$cond:[{$and:[{$gte:['$zf',500]},{$lt:['$zf',600]}]},'良','差']} ]}}})`
> 输出结果：
```
{ "_id" : ObjectId("5767a9d4f4de94710e104213"), "name" : "mingren", "zf" : 325, "flag" : "差" }
{ "_id" : ObjectId("5767a97df4de94710e104212"), "name" : "lisi", "zf" : 579, "flag" : "良" }
{ "_id" : ObjectId("5767a96df4de94710e104211"), "name" : "lisan", "zf" : 579, "flag" : "良" }
{ "_id" : ObjectId("5767a959f4de94710e104210"), "name" : "lisan", "zf" : 555, "flag" : "良" }
{ "_id" : ObjectId("5767a936f4de94710e10420f"), "name" : "jiaoxinkang", "zf" : 624, "flag" : "优秀" }
{ "_id" : ObjectId("5767a91ef4de94710e10420e"), "name" : "bochenlong", "zf" : 604, "flag" : "优秀" }
```

##### $ifNull
**语法**
`{ $ifNull: [ <expression>, <replacement-expression-if-null> ] }`
[参见$cond](#cond)

更多聚合管道操作见[官网](https://docs.mongodb.com/manual/reference/operator/aggregation/)

### MongoDB复制
MongoDB复制是将数据同步在多个服务器的过程。复制提供了数据的冗余备份，并在多个服务器上存储数据副本，提高了数据的可用性， 并可以保证数据的安全性。复制还允许您从硬件故障和服务中断中恢复数据。

#### MongoDB复制原理
mongodb的复制至少需要两个节点。其中一个是主节点，负责处理客户端请求，其余的都是从节点，负责复制主节点上的数据。
mongodb各个节点常见的搭配方式为：一主一从、一主多从。
主节点记录在其上的所有操作oplog，从节点定期轮询主节点获取这些操作，然后对自己的数据副本执行这些操作，从而保证从节点的数据与主节点一致。

#### MongoDB复制设置
1. 启动MongoDB `/usr/bin/mongod -f /etc/mongod.conf --replSet rs0`
> 注意，yum安装的MongoDB默认开机启动，使用/etc/mongod.conf，里面设置了MongoDB的数据路径/var/lib/mongo
2. 启动一个副本集 `rs.initiate()`   
```
{
	"info2" : "no configuration specified. Using a default configuration for the set",
	"me" : "localhost.localdomain:27017",
	"ok" : 1
}
```
	相应的命令前出现 `rs0:PRIMARY>`
3. 查看集群配置 `rs.conf()`
```
...
    "protocolVersion" : NumberLong(1),
	"members" : [
		{
			"_id" : 0,
			"host" : "localhost.localdomain:27017",
			"arbiterOnly" : false,
			"buildIndexes" : true,
			"hidden" : false,
			"priority" : 1,
			"tags" : {
				
			},
			"slaveDelay" : NumberLong(0),
			"votes" : 1
		}
	]
...
```
4.  启动MongoDB2 `mongod --port '27018' --dbpath '/data/db1' --replSet rs0`
5.  主库添加子节点 `rs.add('localhost.localdomain:27018')`，注意这里不能用IP
```
{ "ok" : 1 }
```
	查看集群配置 `rs.conf`,节点已经加入
```
...
    "protocolVersion" : NumberLong(1),
     "members" : [
		{
			"_id" : 0,
			"host" : "localhost.localdomain:27017",
			"arbiterOnly" : false,
			"buildIndexes" : true,
			"hidden" : false,
			"priority" : 1,
			"tags" : {
				
			},
			"slaveDelay" : NumberLong(0),
			"votes" : 1
		},
		{
			"_id" : 1,
			"host" : "localhost.localdomain:27018",
			"arbiterOnly" : false,
			"buildIndexes" : true,
			"hidden" : false,
			"priority" : 1,
			"tags" : {
				
			},
			"slaveDelay" : NumberLong(0),
			"votes" : 1
		}
	]
...
```
6.  验证同步
设置从库可读 `db.getMongo().setSlaveOk()`
查看数据库
```
rs0:SECONDARY> show dbs
local  0.000GB
mydb   0.000GB
```
	查看同步到的数据（节点添加即同步主库数据，主库添加数据从库同步）
```
rs0:SECONDARY> db.score.find()
{ "_id" : ObjectId("576920960a90571c780f81e5"), "name" : "wanglei", "team" : 5, "yuwen" : 111, "shuxue" : 121, "yingyu" : 108, "lizong" : [ 80, 76, 77 ] }
```

### MongoDB分片

在Mongodb里面存在另一种集群，就是分片技术,可以满足MongoDB数据量大量增长的需求。
当MongoDB存储海量的数据时，一台机器可能不足以存储数据，也可能不足以提供可接受的读写吞吐量。这时，我们就可以通过在多台机器上分割数据，使得数据库系统能存储和处理更多的数据。

#### MongoDB分片原理
![](http://i.imgur.com/zArMYmD.png)
上图中主要有如下所述三个主要组件：
* Shard:用于存储实际的数据块，实际生产环境中一个shard server角色可由几台机器组个一个relica set承担，防止主机单点故障
* Config Server:mongod实例，存储了整个 ClusterMetadata，其中包括 chunk信息。
* Routers:前端路由，客户端由此接入，且让整个集群看上去像单一数据库，前端应用可以透明使用。

#### MongoDB分片配置
分片实例：
```
shard server 1 : 27020
shard server 2 : 27021
shard server 3 : 27041
shard server 4 : 27042
config server : 27000
route process : 40000
```
1. 启动分片
2. 
