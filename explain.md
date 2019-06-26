
# explain 


## first load data 

```bash
ps -ef |grep mongo |grep -v grep |awk '{print "kill " $2}'
rm -rf /u01/data/mongo01
mkdir --p /u01/data/mongo01

mongod --dbpath /u01/data/mongo01 --logpath /u01/data/mongo01/log.log  --logappend  --oplogSize 10 --smallfiles --fork
```


```bash
mongoimport --drop -d restaurants -c restaurants /u01/demo/restaurants.data
```

## run explain

```js
db.restaurants.explain("executionStats").find(
   { "cuisine": 1, "borough": "Brooklyn"} 
);
```
```json

"executionStats" : {
                "executionSuccess" : true,
                "nReturned" : 0,
                "executionTimeMillis" : 10,
                "totalKeysExamined" : 0,
                "totalDocsExamined" : 25359,
                "executionStages" : {
                        "stage" : "COLLSCAN",
                        "filter" : {

```
## create index 

```js
db.restaurants.createIndex( { cuisine: 1 ,borough: 1} ) 


```
```js
db.restaurants.explain("executionStats").find(
   { "cuisine": 1, "borough": "Brooklyn"} 
);
```

```json 

   "executionStats" : {
                "executionSuccess" : true,
                "nReturned" : 0,
                "executionTimeMillis" : 0,
                "totalKeysExamined" : 0,
                "totalDocsExamined" : 0,
                "executionStages" : {
                        "stage" : "FETCH",
                        "nReturned" : 0,
                        "executionTimeMillisEstimate" : 0,
                        ... 
                     "invalidates" : 0,
                                "keyPattern" : {
                                        "cuisine" : 1,
                                        "borough" : 1
                                },
                                "indexName" : "cuisine_1_borough_1",
                                "isMultiKey" : false,
                                "multiKeyPaths" : {
                                        "cuisine" : [ ],
                                        "borough" : [ ]
                                },        


```