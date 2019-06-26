
# aggregation 

## Run Replication MongoDB
** The $changeStream stage is only supported on replica sets ** 

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


## lets create stock price  stream  
```js

use test
var docs = [
 { ticker: "WIX", price: 140 },
 { ticker: "WIX", price: 150 },
 { ticker: "WIX", price: 160 },
 { ticker: "WIX", price: 130 },
 { ticker: "WIX", price: 150 }
];
db.Stocks.insert(docs)
```


## install monog on local node 
```
mkdir mongo-proj && cd mongo-proj
npm init -y
npm install mongodb --save
```

## js code 

index.js
```js
const mongo = require("mongodb").MongoClient;
mongo.connect("mongodb://localhost:27001/?replicaSet=t").then(client => {
 console.log("Connected to MongoDB server");
 // Select DB and Collection
 const db = client.db("test");
 const collection = db.collection("Stocks");
 pipeline = [
   {
     $match: { "fullDocument.price": { $gte: 250 } }
   }
 ];
 // Define change stream
 const changeStream = collection.watch(pipeline);
 // start listen to changes
 changeStream.on("change", function(event) {
   console.log(JSON.stringify(event));
 });
});

```

* Note 
you can change the value of operationType parameter with following operations to listen for different types of changes in a collection:

   * Insert
   * Replace (Except unique Id)
   * Update
   * Delete
   * Invalidate (Whenever Mongo returns invalid cursor)



## run 

```
node index.js 
```


## insert 
mongo host76:27001

db.Stocks.insert({ ticker: WIX, price: 100 })

db.Stocks.insert({ ticker: WIX, price: 280 })