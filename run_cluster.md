
# Run Cluster 
2 shared X 3 replicaset (you have to have more the 2 or 2 data and 1 addArb... )


* only shard the big collections
* pre-split if bulk loading
* pick shard key with care, they aren't easily changeable
*  be cognizant of monotonically increasing shard keys
* adding a shard is easy but takes time
* use logical server names for config servers
* don't directly talk to anything except mongos except for dba work
*  keep non-mongos processes off of 27017 to avoid mistakes


#clean
ps -ef |grep mongo |grep -v grep |awk '{print "kill " $2}'

rm -rf  /u01/data/mongocluster


#
```sh 
# clean and create 
ps -ef |grep mongo |grep -v grep |awk '{print "kill " $2}'

rm -rf  /u01/data/mongocluster && mkdir -p /u01/data/mongocluster/a{0..2} /u01/data/mongocluster/cfg{0..2} /u01/data/mongocluster/b{0..2} /u01/data/mongocluster/s{0..2} 

## config nodes (defult port is 27019)
mongod --configsvr --replSet config --port 26050 --dbpath /u01/data/mongocluster/cfg0 --smallfiles --oplogSize 50 --logpath /u01/data/mongocluster/cfg2/log.log --logappend --fork

mongod --configsvr --replSet config --port 26051 --dbpath /u01/data/mongocluster/cfg1 --smallfiles --oplogSize 50 --logpath /u01/data/mongocluster/cfg2/log.log --logappend --fork

mongod --configsvr --replSet config --port 26052 --dbpath /u01/data/mongocluster/cfg2 --smallfiles --oplogSize 50 --logpath /u01/data/mongocluster/cfg2/log.log --logappend --fork


## shard serverd data nodes (2718)
mongod --shardsvr --replSet a --port 27000  --dbpath /u01/data/mongocluster/a0 --smallfiles --oplogSize 50 --logpath /u01/data/mongocluster/a0/log.log --logappend --fork
mongod --shardsvr --replSet a --port 27001 --dbpath /u01/data/mongocluster/a1 --smallfiles --oplogSize 50 --logpath /u01/data/mongocluster/a1/log.log --logappend --fork
mongod --shardsvr --replSet a --port 27002 --dbpath /u01/data/mongocluster/a2 --smallfiles --oplogSize 50 --logpath /u01/data/mongocluster/a2/log.log --logappend --fork
#
mongod --shardsvr --replSet b --port 27010 --dbpath /u01/data/mongocluster/b0 --smallfiles --oplogSize 50 --logpath /u01/data/mongocluster/b0/log.log --logappend --fork
mongod --shardsvr --replSet b --port 27011 --dbpath /u01/data/mongocluster/b1 --smallfiles --oplogSize 50 --logpath /u01/data/mongocluster/b1/log.log --logappend --fork
mongod --shardsvr --replSet b --port 27012 --dbpath /u01/data/mongocluster/b2 --smallfiles --oplogSize 50 --logpath /u01/data/mongocluster/b2/log.log --logappend --fork


## mongos (defult 27017)

mongos --configdb "config/host76:26050,host76:26051,host76:26052"  --logpath /u01/data/mongocluster/s0/log.log --logappend  --fork


mongo #conneto to mongos
```

#





```sh
mongo --port 26050
```

```js

rs.status()


cfg = {
  _id: "config",
  configsvr: true,
  members : [
    {_id:0,host:"host76:26050","priority": 100},
    {_id:1,host:"host76:26051","priority":100},
    {_id:2,host:"host76:26052","priority": 100}
  ]
}
rs.initiate(cfg)

//connect to the first replica set mongo
 mongo --port 27000
rs.status()

///then add more rs
cfg = {
  _id: "a",
  members : [
    {_id:0,host:"host76:27000","priority": 100},
    {_id:1,host:"host76:27001","priority":90},
    {_id:2,host:"host76:27002","priority": 80}
  ]
}
rs.initiate(cfg)

```
//
```sh
 mongo --port 27010
```
```js

rs.status()


cfg = {
  _id: "b",
  members : [
    {_id:0,host:"host76:27010","priority": 100},
    {_id:1,host:"host76:27011","priority":90},
    {_id:2,host:"host76:27012","priority": 80}
  ]
}
rs.initiate(cfg)

```
```
mongo 
```
## build shards 

```js
/// run mongo  // will connet to mongos
/// add shard
//sh.addShard("<relica set name>/<one of the hostnames for the replica>")
sh.addShard("a/host76:27001")
sh.addShard("b/host76:27010")
sh.status()


//The default chunk size for a sharded cluster is 64 megabytes
// for this test lets move it to 1M. (in prod pls keep it 64 megabytes)
use config
db.settings.save( { _id:"chunksize", value: 1 } )


/// create db and data
use testdb
t = db.foo

/// 
sh.enableSharding("testdb")
sh.shardCollection("testdb.foo",{_id:1},true)

for(var i = 0 ;i < 200000 ;i++){
  t.insert({x:i,y:3,x:"test"})
}
//enable sharing on db 

sh.status()
t.getShardDistribution()
sh.isBalancerRunning()

```
