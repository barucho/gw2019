
# aggregation 


```sh
ps -ef |grep mongo |grep -v grep |awk '{print "kill " $2}'
rm -rf /u01/data/mongo01
mkdir --p /u01/data/mongo01

mongod --dbpath /u01/data/mongo01 --logpath /u01/data/mongo01/log.log  --logappend  --oplogSize 10 --smallfiles --fork
```
## import data
```bash
## wget http://media.mongodb.org/zips.json 
mongoimport --drop -d pcat -c zipcodes /u01/demo/zips.json


```
## Return States with Populations above 10 Million
* in sql 
```sql
SELECT state, SUM(pop) AS totalPop
FROM zipcodes
GROUP BY state
HAVING totalPop >= (10*1000*1000)
```

* In this example, the aggregation pipeline consists of the $group  stage followed by the $match stage:

```js

use pcat
db.zipcodes.aggregate( [
   { $group: { _id: "$state", totalPop: { $sum: "$pop" } } },
   { $match: { totalPop: { $gte: 10*1000*1000 } } }
] )
```

##  Return Largest and Smallest Cities by State 

```js
use pcat 

db.zipcodes.aggregate( [
   { $group:
      {
        _id: { state: "$state", city: "$city" },
        pop: { $sum: "$pop" }
      }
   },
   { $sort: { pop: 1 } },
   { $group:
      {
        _id : "$_id.state",
        biggestCity:  { $last: "$_id.city" },
        biggestPop:   { $last: "$pop" },
        smallestCity: { $first: "$_id.city" },
        smallestPop:  { $first: "$pop" }
      }
   },

  // the following $project is optional, and
  // modifies the output format.

  { $project:
    { _id: 0,
      state: "$_id",
      biggestCity:  { name: "$biggestCity",  pop: "$biggestPop" },
      smallestCity: { name: "$smallestCity", pop: "$smallestPop" }
    }
  }
] )

```
