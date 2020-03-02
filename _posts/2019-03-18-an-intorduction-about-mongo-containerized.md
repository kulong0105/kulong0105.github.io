---
title: mongo容器化介绍
category: K8S
tags:
- K8S
---


## 背景

本文将对mongo服务容器化进行详细介绍,包括：
- 单节点部署
- 多节点部署
- K8S实现方案
- 功能验证


<!--more-->

## 单节点部署

docker官方提供了mongo镜像，可以直接使用(https://hub.docker.com/_/mongo)，方式如下：
```bash
docker run -p 27017:27017 -d  -v /my/own/datadir:/data/db -e MONGO_INITDB_ROOT_USERNAME=mongoadmin -e MONGO_INITDB_ROOT_PASSWORD=secret mongo
```

说明：关于mongo镜像如何做到通过环境变量MONGO_INITDB_ROOT_USERNAME和MONGO_INITDB_ROOT_PASSWORD来启用mongo授权功能，类似做如下步骤：
- 以非授权方式启动服务
- 创建用户及相关权限
- 以授权方式启动服务

验证脚本如下
```bash
# cat demo_mongo.sh
 #!/bin/bash

 # [https://github.com/docker-library/mongo/blob/40056ae591c1caca88ffbec2a426e4da07e02d57/3.4/docker-entrypoint.sh]
 # [https://docs.mongodb.com/manual/tutorial/enable-authentication/]

 MONGO_USERNAME="admin"
 MONGO_PASSWORD="secret"

 echo "start mongodb without auth ..."
 rm -rf /var/lib/mongodb && mkdir -p /var/lib/mongodb
 mongod --port 27018 --dbpath /var/lib/mongodb >/dev/null &
 mongo_pid="$!"
 sleep 5

 echo "create admin account ..."
 mongo --port 27018 <<-EOF
 use admin
 db.createUser(
 {
 user: "$MONGO_USERNAME",
 pwd: "$MONGO_PASSWORD",
 roles: [

{ role: "root", db: "admin" }

, "readWriteAnyDatabase" ]
 }
 )
 EOF

 echo "shutdown mongodb ..."
 mongod --port 27018 --dbpath /var/lib/mongodb --shutdown >/dev/null
 sleep 5

 echo "start mongodb with auth ..."
 mongod --port 27018 --auth --dbpath /var/lib/mongodb >/dev/null &
 mongo_pid="$!"
 sleep 5

 echo "connect mongodb and run command ..."
 mongo --port 27018 -u "$MONGO_USERNAME" -p "$MONGO_PASSWORD" <<-EOF
 use admin
 show dbs
 EOF

 echo "shutdown mongodb ..."
 sleep 5
 mongod --port 27018 --dbpath /var/lib/mongodb --shutdown >/dev/null
#
```

运行结果如下：
``` bash
# ./demo_mongo.sh
 start mongodb without auth ...
 create admin account ...
 MongoDB shell version v4.0.9
 connecting to: mongodb://127.0.0.1:27018/?gssapiServiceName=mongodb
 Implicit session: session

{ "id" : UUID("fae30a16-1977-4899-83b9-ea39b1cf697e") }

MongoDB server version: 4.0.9
 switched to db admin
 Successfully added user: {
 "user" : "admin",
 "roles" : [

{ "role" : "root", "db" : "admin" }

,
 "readWriteAnyDatabase"
 ]
 }
 bye
 shutdown mongodb ...
 start mongodb with auth ...
 connect mongodb and run command ...
 MongoDB shell version v4.0.9
 connecting to: mongodb://127.0.0.1:27018/?gssapiServiceName=mongodb
 Implicit session: session

{ "id" : UUID("d80546c7-cff7-4282-ba5c-5156e1713275") }

MongoDB server version: 4.0.9
 switched to db admin
 admin 0.000GB
 config 0.000GB
 local 0.000GB
 bye
 shutdown mongodb ...
#
```


## 多节点部署

### HA方案调查

Mongo HA有三种模式：
- master / slave 模式（不能自动failover）
- replica set (主流，也是官方推荐的）
- sharding (配合replica set一起使用）

技术细节：
- mongo的HA可以不依赖用keepalived，可以通过在应用程序中直接连接多个mongo host，由mongo driver来自动实现选择对应的mongo host
- mongo本身就支持Automatic Failover，不需要做其他操作/配置去支持该特性
mongo 选举算法采用的是bully算法， 与分布式一致性算法raft基本类似，是从raft演变来的，为了避免”脑裂“问题，采用奇数个数据实例(两个节点的副本集不具备真正的故障转移能力，至少需要三个）
- mongo HA支持读写分离，可以设置master节点只写，worker节点负责响应读请求，提供应用程序性能（mongo会自动进行failover，默认2秒进行一次心跳，超过10秒没有回应，即表明master挂掉）
- mongo 提供五种读取策略： 将采用secondaryPreferred策略：优先从 secondary 节点进行读取操作，secondary 节点不可用时从主节点读取数据

### HA方案验证

- 环境搭建步骤

```bash
[root@skyaxe-app-1 ~]# docker run -p 27017:27017 --hostname mongo1 --name mongo -d  mongo:3.6.12 --replSet rs1
651ce0cab92aa32825c40aa41a5cd3bbe4b5b8ab54e511265ad180ad635d5b54
[root@skyaxe-app-1 ~]# docker exec -ti mongo /bin/bash
root@mongo1:/# mongo
MongoDB shell version v3.6.12
connecting to: mongodb://127.0.0.1:27017/?gssapiServiceName=mongodb
Implicit session: session { "id" : UUID("6be341a1-6fb2-499c-aee3-dc22e16e3f03") }
MongoDB server version: 3.6.12
Welcome to the MongoDB shell.
For interactive help, type "help".
For more comprehensive documentation, see
	http://docs.mongodb.org/
Questions? Try the support group
	http://groups.google.com/group/mongodb-user
Server has startup warnings:
2019-08-02T06:11:58.911+0000 I CONTROL  [initandlisten]
2019-08-02T06:11:58.911+0000 I CONTROL  [initandlisten] ** WARNING: Access control is not enabled for the database.
2019-08-02T06:11:58.911+0000 I CONTROL  [initandlisten] **          Read and write access to data and configuration is unrestricted.
2019-08-02T06:11:58.911+0000 I CONTROL  [initandlisten]
2019-08-02T06:11:58.912+0000 I CONTROL  [initandlisten]
2019-08-02T06:11:58.912+0000 I CONTROL  [initandlisten] ** WARNING: You are running on a NUMA machine.
2019-08-02T06:11:58.912+0000 I CONTROL  [initandlisten] **          We suggest launching mongod like this to avoid performance problems:
2019-08-02T06:11:58.912+0000 I CONTROL  [initandlisten] **              numactl --interleave=all mongod [other options]
2019-08-02T06:11:58.912+0000 I CONTROL  [initandlisten]
2019-08-02T06:11:58.912+0000 I CONTROL  [initandlisten] ** WARNING: /sys/kernel/mm/transparent_hugepage/enabled is 'always'.
2019-08-02T06:11:58.912+0000 I CONTROL  [initandlisten] **        We suggest setting it to 'never'
2019-08-02T06:11:58.912+0000 I CONTROL  [initandlisten]
2019-08-02T06:11:58.912+0000 I CONTROL  [initandlisten] ** WARNING: /sys/kernel/mm/transparent_hugepage/defrag is 'always'.
2019-08-02T06:11:58.912+0000 I CONTROL  [initandlisten] **        We suggest setting it to 'never'
2019-08-02T06:11:58.912+0000 I CONTROL  [initandlisten]
> rs.initiate({_id:"rs",members:[ {_id:0,host:'192.168.50.111:27017'},    {_id:1,host:'192.168.50.121:27017'},     {_id:2,host:'192.168.50.122:27017'}]})
{
	"ok" : 0,
	"errmsg" : "Attempting to initiate a replica set with name rs, but command line reports rs1; rejecting",
	"code" : 93,
	"codeName" : "InvalidReplicaSetConfig"
}
> rs.initiate({_id:"rs1",members:[ {_id:0,host:'192.168.50.111:27017'},    {_id:1,host:'192.168.50.121:27017'},     {_id:2,host:'192.168.50.122:27017'}]})
{
	"ok" : 1,
	"operationTime" : Timestamp(1564726408, 1),
	"$clusterTime" : {
		"clusterTime" : Timestamp(1564726408, 1),
		"signature" : {
			"hash" : BinData(0,"AAAAAAAAAAAAAAAAAAAAAAAAAAA="),
			"keyId" : NumberLong(0)
		}
	}
}
rs1:SECONDARY> rs.status()
{
	"set" : "rs1",
	"date" : ISODate("2019-08-02T06:13:33.537Z"),
	"myState" : 2,
	"term" : NumberLong(0),
	"syncingTo" : "",
	"syncSourceHost" : "",
	"syncSourceId" : -1,
	"heartbeatIntervalMillis" : NumberLong(2000),
	"optimes" : {
		"lastCommittedOpTime" : {
			"ts" : Timestamp(0, 0),
			"t" : NumberLong(-1)
		},
		"appliedOpTime" : {
			"ts" : Timestamp(1564726408, 1),
			"t" : NumberLong(-1)
		},
		"durableOpTime" : {
			"ts" : Timestamp(1564726408, 1),
			"t" : NumberLong(-1)
		}
	},
	"members" : [
		{
			"_id" : 0,
			"name" : "192.168.50.111:27017",
			"health" : 1,
			"state" : 2,
			"stateStr" : "SECONDARY",
			"uptime" : 95,
			"optime" : {
				"ts" : Timestamp(1564726408, 1),
				"t" : NumberLong(-1)
			},
			"optimeDate" : ISODate("2019-08-02T06:13:28Z"),
			"syncingTo" : "",
			"syncSourceHost" : "",
			"syncSourceId" : -1,
			"infoMessage" : "could not find member to sync from",
			"configVersion" : 1,
			"self" : true,
			"lastHeartbeatMessage" : ""
		},
		{
			"_id" : 1,
			"name" : "192.168.50.121:27017",
			"health" : 1,
			"state" : 2,
			"stateStr" : "SECONDARY",
			"uptime" : 5,
			"optime" : {
				"ts" : Timestamp(1564726408, 1),
				"t" : NumberLong(-1)
			},
			"optimeDurable" : {
				"ts" : Timestamp(1564726408, 1),
				"t" : NumberLong(-1)
			},
			"optimeDate" : ISODate("2019-08-02T06:13:28Z"),
			"optimeDurableDate" : ISODate("2019-08-02T06:13:28Z"),
			"lastHeartbeat" : ISODate("2019-08-02T06:13:33.103Z"),
			"lastHeartbeatRecv" : ISODate("2019-08-02T06:13:33.132Z"),
			"pingMs" : NumberLong(0),
			"lastHeartbeatMessage" : "",
			"syncingTo" : "",
			"syncSourceHost" : "",
			"syncSourceId" : -1,
			"infoMessage" : "",
			"configVersion" : 1
		},
		{
			"_id" : 2,
			"name" : "192.168.50.122:27017",
			"health" : 1,
			"state" : 2,
			"stateStr" : "SECONDARY",
			"uptime" : 5,
			"optime" : {
				"ts" : Timestamp(1564726408, 1),
				"t" : NumberLong(-1)
			},
			"optimeDurable" : {
				"ts" : Timestamp(1564726408, 1),
				"t" : NumberLong(-1)
			},
			"optimeDate" : ISODate("2019-08-02T06:13:28Z"),
			"optimeDurableDate" : ISODate("2019-08-02T06:13:28Z"),
			"lastHeartbeat" : ISODate("2019-08-02T06:13:33.103Z"),
			"lastHeartbeatRecv" : ISODate("2019-08-02T06:13:33.132Z"),
			"pingMs" : NumberLong(0),
			"lastHeartbeatMessage" : "",
			"syncingTo" : "",
			"syncSourceHost" : "",
			"syncSourceId" : -1,
			"infoMessage" : "",
			"configVersion" : 1
		}
	],
	"ok" : 1,
	"operationTime" : Timestamp(1564726408, 1),
	"$clusterTime" : {
		"clusterTime" : Timestamp(1564726408, 1),
		"signature" : {
			"hash" : BinData(0,"AAAAAAAAAAAAAAAAAAAAAAAAAAA="),
			"keyId" : NumberLong(0)
		}
	}
}
rs1:SECONDARY>
rs1:PRIMARY>
rs1:PRIMARY>
rs1:PRIMARY>
rs1:PRIMARY>
rs1:PRIMAR
```
注：  
1)在另外两台机器也执行同样的命令启动mongo: `docker run -p 27017:27017 --hostname mongo1 --name mongo -d mongo:3.6.12 --replSet rs1`  
2)在进行初始化的时候_id, 必须为启动mongo服务--replSet所指定的参数  
3)在初始化完成后，这台初始化的实例会变为PRIMARY节点（刚开始为SECONDARY，继续按下Enter按键就会发现其变为PRIMARY了）  

- 验证数据同步
在Primary节点插入数据
```bash
rs1:PRIMARY> show dbs
admin   0.000GB
config  0.000GB
local   0.000GB
rs1:PRIMARY> use testdb
switched to db testdb
rs1:PRIMARY> db.tc.insert({name: "allen"})
WriteResult({ "nInserted" : 1 })
rs1:PRIMARY> db.tc.insert({age: 18})
WriteResult({ "nInserted" : 1 })
rs1:PRIMARY> show dbs
admin   0.000GB
config  0.000GB
local   0.000GB
testdb  0.000GB
rs1:PRIMARY> show collections
tc
rs1:PRIMARY> db.tc.find()
{ "_id" : ObjectId("5d43d71d2449aa97c0c85756"), "name" : "allen" }
{ "_id" : ObjectId("5d43d7282449aa97c0c85757"), "age" : 18 }
rs1:PRIMARY>
```

- 在SECONDARY节点查看数据是否被同步

```bash
rs1:SECONDARY> show dbs
2019-08-02T06:26:18.113+0000 E QUERY    [thread1] Error: listDatabases failed:{
	"operationTime" : Timestamp(1564727171, 1),
	"ok" : 0,
	"errmsg" : "not master and slaveOk=false",
	"code" : 13435,
	"codeName" : "NotMasterNoSlaveOk",
	"$clusterTime" : {
		"clusterTime" : Timestamp(1564727171, 1),
		"signature" : {
			"hash" : BinData(0,"AAAAAAAAAAAAAAAAAAAAAAAAAAA="),
			"keyId" : NumberLong(0)
		}
	}
} :
_getErrorWithCode@src/mongo/shell/utils.js:25:13
Mongo.prototype.getDBs@src/mongo/shell/mongo.js:67:1
shellHelper.show@src/mongo/shell/utils.js:860:19
shellHelper@src/mongo/shell/utils.js:750:15
@(shellhelp2):1:1
rs1:SECONDARY> rs.slaveOk()
rs1:SECONDARY> show dbs
admin   0.000GB
config  0.000GB
local   0.000GB
testdb  0.000GB
rs1:SECONDARY> use testdb
switched to db testdb
rs1:SECONDARY> show collections
tc
rs1:SECONDARY> db.tc.find()
{ "_id" : ObjectId("5d43d71d2449aa97c0c85756"), "name" : "allen" }
{ "_id" : ObjectId("5d43d7282449aa97c0c85757"), "age" : 18 }
rs1:SECONDARY> db.tc.insert({location: "Nanjing"})
WriteResult({ "writeError" : { "code" : 10107, "errmsg" : "not master" } })
rs1:SECONDARY>
```
注：  
1)可以发现，默认SECONDARY节点没有读权限，需要调用 rs.slaveOk()来设置  
2)SECONDARY节点没有写权限，只有读权限  


- 自动failover验证
关闭Primary节点的mongo服务
```bash
[root@skyaxe-app-1 ~]# docker stop mongo && docker rm mongo
mongo
mongo
[root@skyaxe-app-1 ~]#
```


- 在SECONDARY节点查看

```
rs1:SECONDARY>
rs1:PRIMARY> rs.status()
{
	"set" : "rs1",
	"date" : ISODate("2019-08-02T06:31:05.848Z"),
	"myState" : 1,
	"term" : NumberLong(2),
	"syncingTo" : "",
	"syncSourceHost" : "",
	"syncSourceId" : -1,
	"heartbeatIntervalMillis" : NumberLong(2000),
	"optimes" : {
		"lastCommittedOpTime" : {
			"ts" : Timestamp(1564727459, 1),
			"t" : NumberLong(2)
		},
		"readConcernMajorityOpTime" : {
			"ts" : Timestamp(1564727459, 1),
			"t" : NumberLong(2)
		},
		"appliedOpTime" : {
			"ts" : Timestamp(1564727459, 1),
			"t" : NumberLong(2)
		},
		"durableOpTime" : {
			"ts" : Timestamp(1564727459, 1),
			"t" : NumberLong(2)
		}
	},
	"members" : [
		{
			"_id" : 0,
			"name" : "192.168.50.111:27017",
			"health" : 0,
			"state" : 8,
			"stateStr" : "(not reachable/healthy)",                 <===
			"uptime" : 0,
			"optime" : {
				"ts" : Timestamp(0, 0),
				"t" : NumberLong(-1)
			},
			"optimeDurable" : {
				"ts" : Timestamp(0, 0),
				"t" : NumberLong(-1)
			},
			"optimeDate" : ISODate("1970-01-01T00:00:00Z"),
			"optimeDurableDate" : ISODate("1970-01-01T00:00:00Z"),
			"lastHeartbeat" : ISODate("2019-08-02T06:31:05.685Z"),
			"lastHeartbeatRecv" : ISODate("2019-08-02T06:30:47.898Z"),
			"pingMs" : NumberLong(0),
			"lastHeartbeatMessage" : "Connection refused",
			"syncingTo" : "",
			"syncSourceHost" : "",
			"syncSourceId" : -1,
			"infoMessage" : "",
			"configVersion" : -1
		},
		{
			"_id" : 1,
			"name" : "192.168.50.121:27017",
			"health" : 1,
			"state" : 1,
			"stateStr" : "PRIMARY",                 <===
			"uptime" : 1143,
			"optime" : {
				"ts" : Timestamp(1564727459, 1),
				"t" : NumberLong(2)
			},
			"optimeDate" : ISODate("2019-08-02T06:30:59Z"),
			"syncingTo" : "",
			"syncSourceHost" : "",
			"syncSourceId" : -1,
			"infoMessage" : "could not find member to sync from",
			"electionTime" : Timestamp(1564727457, 1),
			"electionDate" : ISODate("2019-08-02T06:30:57Z"),
			"configVersion" : 1,
			"self" : true,
			"lastHeartbeatMessage" : ""
		},
		{
			"_id" : 2,
			"name" : "192.168.50.122:27017",
			"health" : 1,
			"state" : 2,
			"stateStr" : "SECONDARY",
			"uptime" : 1055,
			"optime" : {
				"ts" : Timestamp(1564727459, 1),
				"t" : NumberLong(2)
			},
			"optimeDurable" : {
				"ts" : Timestamp(1564727459, 1),
				"t" : NumberLong(2)
			},
			"optimeDate" : ISODate("2019-08-02T06:30:59Z"),
			"optimeDurableDate" : ISODate("2019-08-02T06:30:59Z"),
			"lastHeartbeat" : ISODate("2019-08-02T06:31:05.682Z"),
			"lastHeartbeatRecv" : ISODate("2019-08-02T06:31:04.309Z"),
			"pingMs" : NumberLong(0),
			"lastHeartbeatMessage" : "",
			"syncingTo" : "192.168.50.121:27017",
			"syncSourceHost" : "192.168.50.121:27017",
			"syncSourceId" : 1,
			"infoMessage" : "",
			"configVersion" : 1
		}
	],
	"ok" : 1,
	"operationTime" : Timestamp(1564727459, 1),
	"$clusterTime" : {
		"clusterTime" : Timestamp(1564727459, 1),
		"signature" : {
			"hash" : BinData(0,"AAAAAAAAAAAAAAAAAAAAAAAAAAA="),
			"keyId" : NumberLong(0)
		}
	}
}
rs1:PRIMARY> db.tc.insert({location: "Nanjing"})                                      <===
WriteResult({ "nInserted" : 1 })
rs1:PRIMARY> db.tc.find()
{ "_id" : ObjectId("5d43d71d2449aa97c0c85756"), "name" : "allen" }
{ "_id" : ObjectId("5d43d7282449aa97c0c85757"), "age" : 18 }
{ "_id" : ObjectId("5d43d942c7952a32106c0b3d"), "location" : "Nanjing" }
rs1:PRIMARY>
```

- 启动原Primary节点的mongo服务

```bash
[root@skyaxe-app-1 ~]# docker run -p 27017:27017 --hostname mongo1 --name mongo -d  mongo:3.6.12 --replSet rs1
4cdc27d904fe9af6a2fd875a920d3b4550ddd249b513e31855ed5a7d0f7eee75
[root@skyaxe-app-1 ~]# docker exec -ti mongo /bin/bash
root@mongo1:/# mongo
MongoDB shell version v3.6.12
connecting to: mongodb://127.0.0.1:27017/?gssapiServiceName=mongodb
Implicit session: session { "id" : UUID("e05fc09b-c3e4-4f38-a06f-5101dc0d1983") }
MongoDB server version: 3.6.12
Welcome to the MongoDB shell.
For interactive help, type "help".
For more comprehensive documentation, see
	http://docs.mongodb.org/
Questions? Try the support group
	http://groups.google.com/group/mongodb-user
Server has startup warnings:
2019-08-02T06:34:03.690+0000 I CONTROL  [initandlisten]
2019-08-02T06:34:03.690+0000 I CONTROL  [initandlisten] ** WARNING: Access control is not enabled for the database.
2019-08-02T06:34:03.690+0000 I CONTROL  [initandlisten] **          Read and write access to data and configuration is unrestricted.
2019-08-02T06:34:03.690+0000 I CONTROL  [initandlisten]
2019-08-02T06:34:03.690+0000 I CONTROL  [initandlisten]
2019-08-02T06:34:03.690+0000 I CONTROL  [initandlisten] ** WARNING: You are running on a NUMA machine.
2019-08-02T06:34:03.690+0000 I CONTROL  [initandlisten] **          We suggest launching mongod like this to avoid performance problems:
2019-08-02T06:34:03.690+0000 I CONTROL  [initandlisten] **              numactl --interleave=all mongod [other options]
2019-08-02T06:34:03.690+0000 I CONTROL  [initandlisten]
2019-08-02T06:34:03.690+0000 I CONTROL  [initandlisten] ** WARNING: /sys/kernel/mm/transparent_hugepage/enabled is 'always'.
2019-08-02T06:34:03.690+0000 I CONTROL  [initandlisten] **        We suggest setting it to 'never'
2019-08-02T06:34:03.691+0000 I CONTROL  [initandlisten]
2019-08-02T06:34:03.691+0000 I CONTROL  [initandlisten] ** WARNING: /sys/kernel/mm/transparent_hugepage/defrag is 'always'.
2019-08-02T06:34:03.691+0000 I CONTROL  [initandlisten] **        We suggest setting it to 'never'
2019-08-02T06:34:03.691+0000 I CONTROL  [initandlisten]
rs1:SECONDARY> rs.slaveOk()
rs1:SECONDARY> use testdb
switched to db testdb
rs1:SECONDARY> db.tc.find()
{ "_id" : ObjectId("5d43d71d2449aa97c0c85756"), "name" : "allen" }
{ "_id" : ObjectId("5d43d7282449aa97c0c85757"), "age" : 18 }
{ "_id" : ObjectId("5d43d942c7952a32106c0b3d"), "location" : "Nanjing" }
rs1:SECONDARY>
```
注：  
1)在原Primary节点恢复后，变为SECONDARY节点了，没有进行抢占变为Primary节点  
2)在原Primary节点恢复后，自动从新Primary同步新数据了  
3)如果所有的Secondary都宕机了，只剩下Primary节点，Primary会变成Secondary，不能提供服务(只能读不能写）  

注意事项：
- 上面的验证没有enable auth，参考如下链接验证了enable auth之后的HA功能：(https://medium.com/@ManagedKube/deploy-a-mongodb-cluster-in-steps-9-using-docker-49205e231319)
- keyfile必须为600权限，否则mongo会报“permission too open”错误
- container里面的mongo uid和gid都为999，所以需要在host上使用*chown 999:999*修改uid/gid为999(不能使用*chown mongo:mongo*)
- 由于mongo的image实现了如果使用了MONGO_INITDB_ROOT_USERNAME和MONGO_INITDB_ROOT_PASSWORD环境变量的话，会自动创建mongo的admin用户，所以不必再手工调用mongo命令去创建，具体做法：
    - 先挂载mongo数据目录到container里面，然后使用带环境变量的方式启动，这个时候mongo container会自动创建admin用户，然后关闭container
    - 使用同样的mongo数据目录挂载到container里面，这个时候不要再使用环境变量的方式来启动container了
- mongo image创建用户的时候，是切换到admin db下才创建的，所以我们如果要手工创建用户时，也要先切换到admin db之后再创建。同样，在调用db.auth()进行授权的时候，也要先切换到admin db，否则会授权失败。


##  K8S实现方案

相比采用Deployment，使用StatefulSet会遇到如下挑战/调整：
- 采用StatefulSet来实现cluster级别的服务，其所采用的存储都是基于PV来做，即通过设置volumeClaimTemplates来使得K8S自动为每个pod分配一个PV，而当前的K8S系统没有配置任何PV/PVC （https://kubernetes.io/docs/concepts/workloads/controllers/statefulset/#components)
- 由于采用replicas等于3，如何确保每个pod运行在不同的应用节点
- 使用DNS的方式访问mongo服务，不再使用service ip的方式去访问

针对如上问题，采用如下方法/技术/特性去解决：
- 在3个应用节点配置完全一致的mongo挂载目录（如：/SkyDiscovery/mongo)
- 采用K8S的podAntiAffinity特性去确保每个应用节点只运行一个mongo pod
(https://kubernetes.io/docs/concepts/configuration/assign-pod-node/#affinity-and-anti-affinity)
- 将采用多个新特性确保statefulset工作正常：
    - 确保每个pod是按照顺序启动的: `PodManagementPolicy: OrderedReady`
    - 不管pod的liveness检查是否成功，先更新DNS的SRV记录，确保各个mongo服务能够通过 subdomain 进行互相访问，从而进行集群初始化`publishNotReadyAddresses: true`
    - 固定mongo的servicename为mong: `serviceName: mongo`


在使用K8S实现mongo HA遇到的问题以及思路/解决方法记录：
- 问题：如何初始化集群？
- 思路1：利用InitContainer进行初始化集群，但会有两个问题：
    - 此时集群没有起来，无法使用subdomain在各个实例间进行通信
    - 初始化集群只有执行一次，放在InitContainer中每次重新pod都会执行
- 思路2：在部署代码中先使用docker方式进行集群初始化，然后再使用K8S方式启动mongo服务，但这会有个问题：
    - 在使用docker方法进行集群初始化时只能使用IP方式，因为此时没有pod，无法使用subdomain进行初始化
- 解决方法：
    - 使用环境变量的方式以K8S形式把各个mongo服务先启动起来
    - 在部署代码中，调用`kubectl exec`进入其中一个mongo pod进行集群初始化


- 问题： 如何使用keyfile文件？
- 思路：使用secret或者configmap(这种方式最优雅)，但遇到两个问题：
    - 由于mongo对keyfile文件的权限有明确要求(600)，使用secret或者configmap虽然可以这是文件权限，但是由于不能设置文件owner，导致虽然以600权限map到pod里面，但是mongo服务没有权限读取该文件
    - 使用pod的PreStart特性，在container启动前，先copy已map到container里面的文件，然后修改文件权限和owner，但是由于runc存在一个bug，导致PreStart特性无法在Host的SELinux关闭状态下使用（https://github.com/opencontainers/runc/issues/1343#issuecomment-282438289）
- 解决方法：使用hostPath的方式映射keyfile到pod里面，但是在map前，需要确保每个keyfile文件的权限和owner


## 常用命令

- rs.initiate() # 初始化rs
- rs.status() # 查看集群状态
- rs.config() # 查看集群配置
- rs.reconfig(cfg) # 修改集群配置，需要先执行如： cfg=rs.conf() && cfg.members[1].priority=2
- rs.add() # 增加节点到集群
- rs.remove() # 从集群中删除节点
- rs.slaveOk() # 允许读操作
- rs.stepDown() # 从Primary节点降级为Secondary节点
- rs.printReplicationInfo() # 打印同步信息
- rs.printSlaveReplicationInfo() # 打印slave信息


## python连接

- 无密码：使用无密码配置的时候，使用python链接mongo，可以采用如下方式：

```
from pymodm.connection import connect
connect("mongodb://@localhost:27017/myDatabase)
```

- 有密码：mongo配置了密码后，根据pymodm API文档，尝试使用如下方式:

```
from pymodm.connection import connect
connect("mongodb://@localhost:27017/myDatabase, username=xxx, password=yyy)
```
说明：在进行写myDatabase时，会报权限相关错误。在mongo下使用root权限的用户/密码默认连接admin database（详细可查看：https://github.com/docker-library/mongo/issues/366）


- 有密码：通过调查发现如下方式可以直接利用admin进行授权，不必事先针对每个database进行单独授权：

```
from pymodm.connection import connect
connect("mongodb://$USERNAME:$PASSWORD@localhost:27017/$DATABASE_NAME?authSource=admin")
```
参考：
- https://pymodm.readthedocs.io/en/0.4.1/api/index.html#
- https://api.mongodb.com/python/current/examples/authentication.html
- https://stackoverflow.com/questions/52741957/authentication-failed-error-in-python-script-to-connect-mongodb-server-using-pym

## 功能验证

### 命令行

```bash
[root@lx-app-0 auto-deploy]# k exec -ti mongo-0 -n skydiscovery /bin/bash
[root@mongo-0 /]# mongo
MongoDB shell version: 3.2.4
connecting to: test
Welcome to the MongoDB shell.
For interactive help, type "help".
For more comprehensive documentation, see
	http://docs.mongodb.org/
Questions? Try the support group
	http://groups.google.com/group/mongodb-user
skydiscovery-rs:PRIMARY> use admin
switched to db admin
skydiscovery-rs:PRIMARY> rs.status
function () { return db._adminCommand("replSetGetStatus"); }
skydiscovery-rs:PRIMARY> rs.status()
{
	"ok" : 0,
	"errmsg" : "not authorized on admin to execute command { replSetGetStatus: 1.0 }",
	"code" : 13
}
skydiscovery-rs:PRIMARY> db.auth()
Error: auth expects either (username, password) or ({ user: username, pwd: password })
0
skydiscovery-rs:PRIMARY> rs.status(admin, skydata@1229)
2019-11-05T08:23:14.988+0000 E QUERY    [thread1] SyntaxError: illegal character @(shell):1:24

skydiscovery-rs:PRIMARY> db.auth(admin,skydata@1229)
2019-11-05T08:23:30.912+0000 E QUERY    [thread1] SyntaxError: illegal character @(shell):1:21

skydiscovery-rs:PRIMARY> db.auth("admin", "skydata@1229")
1
skydiscovery-rs:PRIMARY> db.status()
2019-11-05T08:23:49.356+0000 E QUERY    [thread1] TypeError: db.status is not a function :
@(shell):1:1

skydiscovery-rs:PRIMARY> rs.status()
{
	"set" : "skydiscovery-rs",
	"date" : ISODate("2019-11-05T08:23:54.505Z"),
	"myState" : 1,
	"term" : NumberLong(1),
	"heartbeatIntervalMillis" : NumberLong(2000),
	"members" : [
		{
			"_id" : 0,
			"name" : "mongo-0.mongo-headless.skydiscovery.svc.cluster.local:27017",
			"health" : 1,
			"state" : 1,
			"stateStr" : "PRIMARY",
			"uptime" : 1576,
			"optime" : {
				"ts" : Timestamp(1572940667, 1),
				"t" : NumberLong(1)
			},
			"optimeDate" : ISODate("2019-11-05T07:57:47Z"),
			"electionTime" : Timestamp(1572940666, 2),
			"electionDate" : ISODate("2019-11-05T07:57:46Z"),
			"configVersion" : 1,
			"self" : true
		}
	],
	"ok" : 1
}
skydiscovery-rs:PRIMARY> show dbs
admin  0.078GB
local  1.078GB
skydiscovery-rs:PRIMARY> use testdb
switched to db testdb
skydiscovery-rs:PRIMARY> db.tc.insert({name: "allen"})
WriteResult({ "nInserted" : 1 })
skydiscovery-rs:PRIMARY> rs.status()
{
	"set" : "skydiscovery-rs",
	"date" : ISODate("2019-11-05T08:35:37.473Z"),
	"myState" : 1,
	"term" : NumberLong(1),
	"heartbeatIntervalMillis" : NumberLong(2000),
	"members" : [
		{
			"_id" : 0,
			"name" : "mongo-0.mongo-headless.skydiscovery.svc.cluster.local:27017",
			"health" : 1,
			"state" : 1,
			"stateStr" : "PRIMARY",
			"uptime" : 2279,
			"optime" : {
				"ts" : Timestamp(1572942302, 2),
				"t" : NumberLong(1)
			},
			"optimeDate" : ISODate("2019-11-05T08:25:02Z"),
			"electionTime" : Timestamp(1572940666, 2),
			"electionDate" : ISODate("2019-11-05T07:57:46Z"),
			"configVersion" : 1,
			"self" : true
		}
	],
	"ok" : 1
}
skydiscovery-rs:PRIMARY> use testdb
switched to db testdb
skydiscovery-rs:PRIMARY> db.tc.insert({name: "allen0105"})
WriteResult({ "nInserted" : 1 })
skydiscovery-rs:PRIMARY> show dbs
admin   0.078GB
local   1.078GB
testdb  0.078GB
skydiscovery-rs:PRIMARY> show collections
system.indexes
tc
skydiscovery-rs:PRIMARY> db.tc.find()
{ "_id" : ObjectId("5dc131de56cde07237e72055"), "name" : "allen" }
{ "_id" : ObjectId("5dc1346c19722b1422693d4d"), "name" : "allen0105" }
skydiscovery-rs:PRIMARY>
```


### python

```
In [50]: from pymongo import MongoClient
In [51]: from datetime import datetime
In [52]: import time
In [53]: c = MongoClient('mongodb://mongoadmin:secret@192.168.50.111:27017,192.168.50.121:27017,192.168.50.122:27017/testdb?replicaSet=rs1&authSource=admin&readPreference=secondaryPreferred')
In [54]: def mongo_ha_test(start, end):
    ...:     print(str(datetime.now()))
    ...:
    ...:     for i in range(start,end):
    ...:         print(i)
    ...:         time.sleep(1)
    ...:         c.testdb.tc.insert_one({"x": i})
    ...:
    ...:     print(str(datetime.now()))

In [55]: mongo_ha_test(10,20)
2019-08-03 14:40:33.358706
10
11
12
13
14
15
16
17
18
19
2019-08-03 14:40:43.498458        <=== 总共花了10s
In [56]: mongo_ha_test(20,30)
2019-08-03 14:41:30.245332
20
21
22
23            <=== 在这个点中断Primary节点的mongo服务，然后输出"暂停”了
24
25
26
27
28
29
2019-08-03 14:41:51.805824      <=== 总共花了20s
In [57]:
```
说明：
- 连接Mongo时使用secondaryPreferred参数，表明优先从secondary节点进行读操作
- mongo_ha_test(10,20)的输出表明数据按照预期写入到了数据库
- mongo_ha_test(20,30)在运行过程中，中断Primary节点的mongo服务，可以看出最终所有数据还是写到mongo服务，但是总共花了20s，非10s。这是因为mongo的primary节点挂掉后，集群默认设置的failover timeout是10s，也就是10s之后才能选举出新的Primary节点
- 如果在读操作过程中，Primary节点挂掉了，由于设置了secondaryPreferred策略，读取数据不会有任何间断


## FAQ

在非HA部署模式下，mongo没有创建replicaSet，会使得python使用如下方式无法连接mongo
```
from pymodm.connection import connect
connect("mongodb://$USERNAME:$PASSWORD@$MONGO_HOST_PORT/$DATABASE_NAME?authSource=admin&replicaSet=skydiscovery-rs&readPreference=secondaryPreferred")
```

