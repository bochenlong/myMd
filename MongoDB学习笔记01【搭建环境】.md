### MongoDB简介
MongoDB 是由C++语言编写的，是一个基于分布式文件存储的开源数据库系统。MongoDB 将数据存储为一个文档，数据结构由键值(key=>value)对组成。MongoDB 文档类似于 JSON 对象。字段值可以包含其他文档，数组及文档数组。
### MongoDB特点
* MongoDB的提供了一个面向文档存储，操作起来比较简单和容易。
* 你可以在MongoDB记录中设置任何属性的索引 (如：FirstName="Sameer",Address="8 Gandhi Road")来实现更快的排序。
* 你可以通过本地或者网络创建数据镜像，这使得MongoDB有更强的扩展性。
* 如果负载的增加（需要更多的存储空间和更强的处理能力） ，它可以分布在计算机网络中的其他节点上这就是所谓的分片。
* Mongo支持丰富的查询表达式。查询指令使用JSON形式的标记，可轻易查询文档中内嵌的对象及数组。
* MongoDb 使用update()命令可以实现替换完成的文档（数据）或者一些指定的数据字段 。
* Mongodb中的Map/reduce主要是用来对数据进行批量处理和聚合操作。
* Map和Reduce。Map函数调用emit(key,value)遍历集合中所有的记录，将key与value传给Reduce函数进行处理。
* Map函数和Reduce函数是使用Javascript编写的，并可以通过db.runCommand或mapreduce命令来执行MapReduce操作。
* GridFS是MongoDB中的一个内置功能，可以用于存放大量小文件。
* MongoDB允许在服务端执行脚本，可以用Javascript编写某个函数，直接在服务端执行，也可以把函数的定义存储在服务端，下次直接调用即可。
* MongoDB支持各种编程语言:RUBY，PYTHON，JAVA，C++，PHP，C#等多种语言。
* MongoDB安装简单。

### MongoDB安装
#### 安装
*系统环境* `CentOS release 6.6 (Final) `
<!-- more -->
*软件安装*，参照官网使用 **yum** 方式安装
1 配置 **yum** 源   
```
$ touch /etc/yum.repos.d/mongodb-org-3.2.repo
$ vi /etc/yum.repos.d/mongodb-org-3.2.repo
```
> [mongodb-org-2.6]
  name=MongoDB 2.6 Repository
  baseurl=http://downloads-distro.mongodb.org/repo/redhat/os/x86_64/
  gpgcheck=0
  enabled=1
2 安装MongoDB
```
$ sudo yum install -y mongodb-org
或者自定义
$ sudo yum install -y mongodb-org-3.2.7 mongodb-org-server-3.2.7 mongodb-org-shell-3.2.7 mongodb-org-mongos-3.2.7 mongodb-org-tools-3.2.7
```

#### 启动
首先，需要手动创建MongoDB的数据库目录，默认为`/data/db`：
```
$ mkdir /data/db
$ mongod
```
亦可以通过指定目录来运行MongoDB：
```
$ mongod --dbpath
```
需要注意，MongoDB官方建议我们使用`非root`用户且关闭`transparent_hugepage`：
```
2016-06-15T11:42:34.371+0800 I CONTROL  [initandlisten] ** WARNING: You are running this process as the root user, which is not recommended.
2016-06-15T11:42:34.371+0800 I CONTROL  [initandlisten] ** WARNING: /sys/kernel/mm/transparent_hugepage/enabled is 'always'.
2016-06-15T11:42:34.371+0800 I CONTROL  [initandlisten] **        We suggest setting it to 'never'
2016-06-15T11:42:34.371+0800 I CONTROL  [initandlisten] 
2016-06-15T11:42:34.371+0800 I CONTROL  [initandlisten] ** WARNING: /sys/kernel/mm/transparent_hugepage/defrag is 'always'.
2016-06-15T11:42:34.371+0800 I CONTROL  [initandlisten] **        We suggest setting it to 'never'

```
第一个问题，MongoDB在 **yum** 安装的时候会自动创建 **mongod** 用户，因此我们使用 **mongod**用户启动：
```
$ cat /etc/passwd 
> mongod:x:497:496:mongod:/var/lib/mongo:/bin/false

$ cp /etc/skel/.bash_logout .bash_profile .bashrc /home/mongod
> /var/lib/mongo 为mongod用户的根目录，初始化用户根目录，否则切换用户出现$ -bash-4.1

$ vi /etc/passwd 
> /bin/false是最严格的禁止login选项，一切服务都不能用。修改/bin/false为/bin/bash

$ su - mongod 
$ mongod
```
第二、三个问题，Mongod官方建议关闭Linux的内存巨页（脚本方式）：
**1 创建开机启动服务**
```
$ touch /etc/init.d/disable-transparent-hugepages
$ vi /etc/init.d/disable-transparent-hugepages

```
```
#!/bin/sh
### BEGIN INIT INFO
# Provides:          disable-transparent-hugepages
# Required-Start:    $local_fs
# Required-Stop:
# X-Start-Before:    mongod mongodb-mms-automation-agent
# Default-Start:     2 3 4 5
# Default-Stop:      0 1 6
# Short-Description: Disable Linux transparent huge pages
# Description:       Disable Linux transparent huge pages, to improve
#                    database performance.
### END INIT INFO

case $1 in
  start)
    if [ -d /sys/kernel/mm/transparent_hugepage ]; then
      thp_path=/sys/kernel/mm/transparent_hugepage
    elif [ -d /sys/kernel/mm/redhat_transparent_hugepage ]; then
      thp_path=/sys/kernel/mm/redhat_transparent_hugepage
    else
      return 0
    fi

    echo 'never' > ${thp_path}/enabled
    echo 'never' > ${thp_path}/defrag

    unset thp_path
    ;;
esac
```
**2 设置开机启动**

```
$ sudo chmod 755 /etc/init.d/disable-transparent-hugepages
$ 6/16/2016 10:08:56 AM 
```
> 具体也可参见官方 https://docs.mongodb.com/manual/tutorial/transparent-huge-pages/#configure-thp-tuned

### MongoDB交互
1. MongoDB Shell是MongoDB自带的交互式Javascript shell，用来对MongoDB进行操作和管理的交互式环境。
```
[root@localhost ~]# mongo
MongoDB shell version: 3.2.7
connecting to: test
>
```
亦可以通过 `mongodb://username:password@hostname/dbname`，比如：
```
// 链接本地数据库
mongodb://localhost
// 安全模式链接到本地数据库
mongodb://localhost/?safe=true
// 直接连接第一个服务器，无论是replica set一部分或者主服务器或者从服务器。
mongodb://host1,host2,host3/?connect=direct;slaveOk=true
```

参数选项说明：

选项|描述
---|---
replicaSet=name	|验证replica set的名称。 Impliesconnect=replicaSet.
slaveOk=true/false	|true:在connect=direct模式下，驱动会连接第一台机器，即使这台服务器不是主。在connect=replicaSet模式下，驱动会发送所有的写请求到主并且把读取操作分布在其他从服务器。false: 在 connect=direct模式下，驱动会自动找寻主服务器. 在connect=replicaSet 模式下，驱动仅仅连接主服务器，并且所有的读写命令都连接到主服务器。
safe=true/false | true: 在执行更新操作之后，驱动都会发送getLastError命令来确保更新成功。(还要参考 wtimeoutMS). false: 在每次更新之后，驱动不会发送getLastError来确保更新成功。
w=n	| 驱动添加 { w : n } 到getLastError命令. 应用于safe=true。
wtimeoutMS=ms|	驱动添加 { wtimeout : ms } 到 getlasterror 命令. 应用于 safe=true.
fsync=true/false |	true: 驱动添加 { fsync : true } 到 getlasterror 命令.应用于 safe=true.  false: 驱动不会添加到getLastError命令中。
journal=true/false|	如果设置为 true, 同步到 journal (在提交到数据库前写入到实体中). 应用于 safe=true
connectTimeoutMS=ms|	可以打开连接的时间。
socketTimeoutMS=ms|	发送和接受sockets的时间。
2. MongoDB 提供了简单的 HTTP 用户界面。 如果你想启用该功能，需要在启动的时候指定参数 --rest 。
![](http://i.imgur.com/6WQwxFw.png)

### MongoDB关闭
从MongoDB的admin中关闭（推荐用这种方法）：
```
$ use admin   
> switched to db admin    
$ db.shutdownServer()    
> server should be down...
```
或者
```
mongod --shutdown
``