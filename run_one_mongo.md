
# run one mongodb instance 

```sh
ps -ef |grep mongo |grep -v grep |awk '{print "kill " $2}'
rm -rf /u01/data/mongo01
mkdir --p /u01/data/mongo01

mongod --dbpath /u01/data/mongo01 --logpath /u01/data/mongo01/log.log  --logappend  --oplogSize 10 --smallfiles --fork


mongoimport --drop -d pcat -c testdb /u01/demo/zips.json
```

