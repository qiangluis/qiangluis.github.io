---
layout:     post
title:      "mongodb 安装与使用令"
subtitle:   "mongodb 安装与使用"
date:       2017-05-09 17:25:00
author:     "LQ"
header-img: "img/in-post/mongodb.jpg"
header-mask: 0.3
catalog:    false
tags:
    - mongodb
---

## mongodb 的安装

```
>> wget mongodb相应系统的链接

>> tar -zxvf 压缩包

>> #配置环境变量
>> vi /etc/profile

```
我这里没有使用mongo.conf,配置文件在启动mongod 的使用要指定位置

启动mongo
```
# 指定data的存放地点和log的存放地点以及开启用户权限
>> mongod -dbpath=/data/mdb --fork --port 27017 --logpath=/opt/logs/mongodb.log --logappend --auth
```
把该命令放到开机启动


```
>> vi /etc/rc.local
```


## mongodb的使用

###### 用户权限

因为开启了数据库用户认证，所以当我们使用show dbs 命令时会发现多了一个admin库，没开认证这个库是不存在的。MongoDB默认是不认证的，默认没有账号，只要能连接上服务就可以对数据库进行各种操作，mongodb认为安全最好的方法就是在一个可信的环境中运行它，保证之后可信的机器才能访问它，注意一点，帐号是跟着库走的，所以在指定库里授权，必须也在指定库里验证(auth)。
    
```
# admin创建一个root用户
> use admin
switched to db admin
> db.createUser({user:'root',pwd:'123456',roles:['userAdminAnyDatabase']});
```
**注意：***：新版的mongodb已经不支持addUser方法了。改成createUser了。*

==createUser==()函数各个字段解释：

**user**：用户名

**pwd**：密码

**cusomData**：为任意内容，例如可以为用户全名介绍；

**roles**：指定用户的角色，可以用一个空数组给新用户设定空角色；在roles字段,可以指定内置角色和用户定义的角色。role里的角色可以选：


```
内置角色：
1. 数据库用户角色：read、readWrite;
2. 数据库管理角色：dbAdmin、dbOwner、userAdmin；
3. 集群管理角色：clusterAdmin、clusterManager、clusterMonitor、hostManager；
4. 备份恢复角色：backup、restore；
5. 所有数据库角色：readAnyDatabase、readWriteAnyDatabase、userAdminAnyDatabase、dbAdminAnyDatabase
6. 超级用户角色：root  
    // 这里还有几个角色间接或直接提供了系统超级用户的访问（dbOwner 、userAdmin、userAdminAnyDatabase）
7. 内部角色：__system
```


```
1. Read：允许用户读取指定数据库
2. readWrite：允许用户读写指定数据库
3. dbAdmin：允许用户在指定数据库中执行管理函数，如索引创建、删除，查看统计或访问system.profile
4. userAdmin：允许用户向system.users集合写入，可以找指定数据库里创建、删除和管理用户
5. clusterAdmin：只在admin数据库中可用，赋予用户所有分片和复制集相关函数的管理权限。
6. readAnyDatabase：只在admin数据库中可用，赋予用户所有数据库的读权限
7. readWriteAnyDatabase：只在admin数据库中可用，赋予用户所有数据库的读写权限
8. userAdminAnyDatabase：只在admin数据库中可用，赋予用户所有数据库的userAdmin权限
9. dbAdminAnyDatabase：只在admin数据库中可用，赋予用户所有数据库的dbAdmin权限。
10. root：只在admin数据库中可用。超级账号，超级权限
```

###### 查看用户

```
> use admin
switched to db admin
> db.auth('root','root')
1
> show users;
{
        "_id" : "admin.root",
        "user" : "root",
        "db" : "admin",
        "roles" : [
                {
                        "role" : "userAdminAnyDatabase",
                        "db" : "admin"
                }
        ]
}
> 
```

###### 创建数据库

```
> use newdatebase
switched to db newdatebase
>
```
此时并没有创建好数据库，需要向数据库中插入一些数据才能在show dbs 显示数据库



```
> db.test.save({username:'lasfsd',pwd:'sdfsd"});
```

###### 一些其他命令
这里就不介绍了，使用help 命令查看mongodb的命令


```
> help
        db.help()                    help on db methods
        db.mycoll.help()             help on collection methods
        sh.help()                    sharding helpers
        rs.help()                    replica set helpers
        help admin                   administrative help
        help connect                 connecting to a db help
        help keys                    key shortcuts
        help misc                    misc things to know
        help mr                      mapreduce

        show dbs                     show database names
        show collections             show collections in current database
        show users                   show users in current database
        show profile                 show most recent system.profile entries with time >= 1ms
        show logs                    show the accessible logger names
        show log [name]              prints out the last segment of log in memory, 'global' is default
        use <db_name>                set current database
        db.foo.find()                list objects in collection foo
        db.foo.find( { a : 1 } )     list objects in foo where a == 1
        it                           result of the last line evaluated; use to further iterate
        DBQuery.shellBatchSize = x   set default number of items to display on shell
        exit                         quit the mongo shell
> db.help()
DB methods:
        db.adminCommand(nameOrDocument) - switches to 'admin' db, and runs command [ just calls db.runCommand(...) ]
        db.auth(username, password)
        db.cloneDatabase(fromhost)
        db.commandHelp(name) returns the help for the command
        db.copyDatabase(fromdb, todb, fromhost)
        db.createCollection(name, { size : ..., capped : ..., max : ... } )
        db.createView(name, viewOn, [ { $operator: {...}}, ... ], { viewOptions } )
        db.createUser(userDocument)
        db.currentOp() displays currently executing operations in the db
        db.dropDatabase()
        db.eval() - deprecated
        db.fsyncLock() flush data to disk and lock server for backups
        db.fsyncUnlock() unlocks server following a db.fsyncLock()
        db.getCollection(cname) same as db['cname'] or db.cname
        db.getCollectionInfos([filter]) - returns a list that contains the names and options of the db's collections
        db.getCollectionNames()
        db.getLastError() - just returns the err msg string
        db.getLastErrorObj() - return full status object
        db.getLogComponents()
        db.getMongo() get the server connection object
        db.getMongo().setSlaveOk() allow queries on a replication slave server
        db.getName()
        db.getPrevError()
        db.getProfilingLevel() - deprecated
        db.getProfilingStatus() - returns if profiling is on and slow threshold
        db.getReplicationInfo()
        db.getSiblingDB(name) get the db at the same server as this one
        db.getWriteConcern() - returns the write concern used for any operations on this db, inherited from server object if set
        db.hostInfo() get details about the server's host
        db.isMaster() check replica primary status
        db.killOp(opid) kills the current operation in the db
        db.listCommands() lists all the db commands
        db.loadServerScripts() loads all the scripts in db.system.js
        db.logout()
        db.printCollectionStats()
        db.printReplicationInfo()
        db.printShardingStatus()
        db.printSlaveReplicationInfo()
        db.dropUser(username)
        db.repairDatabase()
        db.resetError()
        db.runCommand(cmdObj) run a database command.  if cmdObj is a string, turns it into { cmdObj : 1 }
        db.serverStatus()
        db.setLogLevel(level,<component>)
        db.setProfilingLevel(level,<slowms>) 0=off 1=slow 2=all
        db.setWriteConcern( <write concern doc> ) - sets the write concern for writes to the db
        db.unsetWriteConcern( <write concern doc> ) - unsets the write concern for writes to the db
        db.setVerboseShell(flag) display extra information in shell output
        db.shutdownServer()
        db.stats()
        db.version() current version of the server
> 
```

如果想更多的命令，则在集合名后面再加help()

```
> db.test.help()
DBCollection help
        db.test.find().help() - show DBCursor help
        db.test.bulkWrite( operations, <optional params> ) - bulk execute write operations, optional parameters are: w, wtimeout, j
        db.test.count( query = {}, <optional params> ) - count the number of documents that matches the query, optional parameters are: limit, skip, hint, maxTimeMS
        db.test.copyTo(newColl) - duplicates collection by copying all documents to newColl; no indexes are copied.
        db.test.convertToCapped(maxBytes) - calls {convertToCapped:'test', size:maxBytes}} command
        db.test.createIndex(keypattern[,options])
        db.test.createIndexes([keypatterns], <options>)
        db.test.dataSize()
        db.test.deleteOne( filter, <optional params> ) - delete first matching document, optional parameters are: w, wtimeout, j
        db.test.deleteMany( filter, <optional params> ) - delete all matching documents, optional parameters are: w, wtimeout, j
        db.test.distinct( key, query, <optional params> ) - e.g. db.test.distinct( 'x' ), optional parameters are: maxTimeMS
        db.test.drop() drop the collection
        db.test.dropIndex(index) - e.g. db.test.dropIndex( "indexName" ) or db.test.dropIndex( { "indexKey" : 1 } )
        db.test.dropIndexes()
        db.test.ensureIndex(keypattern[,options]) - DEPRECATED, use createIndex() instead
        db.test.explain().help() - show explain help
        db.test.reIndex()
        db.test.find([query],[fields]) - query is an optional query filter. fields is optional set of fields to return.
                                                      e.g. db.test.find( {x:77} , {name:1, x:1} )
        db.test.find(...).count()
        db.test.find(...).limit(n)
        db.test.find(...).skip(n)
        db.test.find(...).sort(...)
        db.test.findOne([query], [fields], [options], [readConcern])
        db.test.findOneAndDelete( filter, <optional params> ) - delete first matching document, optional parameters are: projection, sort, maxTimeMS
        db.test.findOneAndReplace( filter, replacement, <optional params> ) - replace first matching document, optional parameters are: projection, sort, maxTimeMS, upsert, returnNewDocument
        db.test.findOneAndUpdate( filter, update, <optional params> ) - update first matching document, optional parameters are: projection, sort, maxTimeMS, upsert, returnNewDocument
        db.test.getDB() get DB object associated with collection
        db.test.getPlanCache() get query plan cache associated with collection
        db.test.getIndexes()
        db.test.group( { key : ..., initial: ..., reduce : ...[, cond: ...] } )
        db.test.insert(obj)
        db.test.insertOne( obj, <optional params> ) - insert a document, optional parameters are: w, wtimeout, j
        db.test.insertMany( [objects], <optional params> ) - insert multiple documents, optional parameters are: w, wtimeout, j
        db.test.mapReduce( mapFunction , reduceFunction , <optional params> )
        db.test.aggregate( [pipeline], <optional params> ) - performs an aggregation on a collection; returns a cursor
        db.test.remove(query)
        db.test.replaceOne( filter, replacement, <optional params> ) - replace the first matching document, optional parameters are: upsert, w, wtimeout, j
        db.test.renameCollection( newName , <dropTarget> ) renames the collection.
        db.test.runCommand( name , <options> ) runs a db command with the given name where the first param is the collection name
        db.test.save(obj)
        db.test.stats({scale: N, indexDetails: true/false, indexDetailsKey: <index key>, indexDetailsName: <index name>})
        db.test.storageSize() - includes free space allocated to this collection
        db.test.totalIndexSize() - size in bytes of all the indexes
        db.test.totalSize() - storage allocated for all data and indexes
        db.test.update( query, object[, upsert_bool, multi_bool] ) - instead of two flags, you can pass an object with fields: upsert, multi
        db.test.updateOne( filter, update, <optional params> ) - update the first matching document, optional parameters are: upsert, w, wtimeout, j
        db.test.updateMany( filter, update, <optional params> ) - update all matching documents, optional parameters are: upsert, w, wtimeout, j
        db.test.validate( <full> ) - SLOW
        db.test.getShardVersion() - only for use with sharding
        db.test.getShardDistribution() - prints statistics about data distribution in the cluster
        db.test.getSplitKeysForChunks( <maxChunkSize> ) - calculates split points over all chunks and returns splitter function
        db.test.getWriteConcern() - returns the write concern used for any operations on this collection, inherited from server/db if set
        db.test.setWriteConcern( <write concern doc> ) - sets the write concern for writes to the collection
        db.test.unsetWriteConcern( <write concern doc> ) - unsets the write concern for writes to the collection
        db.test.latencyStats() - display operation latency histograms for this collection
>
```












