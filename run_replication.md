# Run Replication MongoDB
starting with 3 nodes 1 arbiter 2 data nodes  



## clean
```sh
# clean 
ps -ef |grep mongo |grep -v grep |awk '{print "kill " $2}'
rm -rf /u01/data


# create 
mkdir -p /u01/data/t{1..4}
```


## run mongod
```sh 
mongod --port 27001 --replSet t --dbpath /u01/data/t1 --logpath /u01/data/t1/log.log  --logappend  --oplogSize 10 --smallfiles --fork
mongod --port 27002 --replSet t --dbpath /u01/data/t2 --logpath /u01/data/t2/log.log  --logappend  --oplogSize 10 --smallfiles --fork
mongod --port 27003 --replSet t --dbpath /u01/data/t3 --logpath /u01/data/t3/log.log  --logappend  --oplogSize 10 --smallfiles --fork
mongod --port 27004 --replSet t --dbpath /u01/data/t4 --logpath /u01/data/t4/log.log  --logappend  --oplogSize 10 --smallfiles --fork

```


## create replica set 
```
mongo host76:27001
```


```js
//create replica config

cfg = {
  _id: "t",
  members : [
    {_id:0,host:"host76:27001","priority": 90},
    {_id:1,host:"host76:27002","arbiterOnly" : true },
    {_id:2,host:"host76:27003","priority": 30}
  ]
}

```

```js
rs.initiate(cfg)

```

```js
// add node
rs.add( { "host": "host76:27004", "priority": 90 ,"buildIndexes" : true,       
         "tags" : {                                                                              
                "use" : "prod",                                                              
                "host" : "host76:27004",                      
                "dc" : "primary"                                                                     
        } } )

```

```js
//rs.help()

rs.initiate(cfg)
rs.status()
rs.reconfig(...)
```
## reconfig 
```js
///prevent 27003 from primary
cfg = rs.conf()
cfg.members[2].priority = 0
rs.reconfig(cfg)

///move  27004 to primary
cfg = rs.conf()
cfg.members[3].priority = 100
rs.reconfig(cfg)



```