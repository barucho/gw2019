# How to install MongoDB on Linux 

- change the ulimit for this user as 
in  /etc/security/limits.conf or if using  the service in the service file :
```
(file size): unlimited
(cpu time): unlimited
(virtual memory): unlimited [1]
(open files): 64000
(memory size): unlimited [1] [2]
(processes/threads): 64000

```

## radhet 
```bash 
vi /etc/yum.repos.d/mongodb-org-4.0.repo
```
[mongodb-org-4.0]
name=MongoDB Repository
baseurl=https://repo.mongodb.org/yum/redhat/$releasever/mongodb-org/4.0/x86_64/
gpgcheck=1
enabled=1
gpgkey=https://www.mongodb.org/static/pgp/server-4.0.asc
```

```bash 
sudo yum install -y mongodb-org
```

## deb 

https://docs.mongodb.com/manual/tutorial/install-mongodb-on-debian/


## Run mongodb

- this is a small test on my laptop so is will run MongoDB stand-alone  
- in the config yaml i change the log path and db path :
- i run --oplogSize 10 reduce the disk space


- if we need to change the data location 
```
mkdir --p /u01/data/mongo01
mkdir --p /u01/log/mongo01
mkdir --p /u01/config/mongo01

cp /etc/mongod.conf /u01/config/mongo01
```
- or just create the file :
```yaml
# mongod.conf

# for documentation of all options, see:
#   http://docs.mongodb.org/manual/reference/configuration-options/

# where to write logging data.
systemLog:
  destination: file
  logAppend: true
  path: /u01/log/mongo01/mongod.log

# Where and how to store data.
storage:
  dbPath: /u01/data/mongo01
  journal:
    enabled: true
#  engine:
#  wiredTiger:

# how the process runs
processManagement:
  fork: true  # fork and run in background
  pidFilePath: /var/run/mongodb/mongod.pid  # location of pidfile
  timeZoneInfo: /usr/share/zoneinfo

# network interfaces
net:
  port: 27017
  bindIp: 127.0.0.1  # Enter 0.0.0.0,:: to bind to all IPv4 and IPv6 addresses or, alternatively, use the net.bindIpAll setting.


#security:

#operationProfiling:

#replication:

#sharding:

## Enterprise-Only Options

#auditLog:

#snmp:


```



## enable service 

```
sudo chkconfig mongod on
```
## start mongodb 

```
sudo service mongod start
```
- or from command line 
```
mongod --config  /u01/config/mongo01/mongod.conf --oplogSize 10 --fork 
```


```bash 
ps aux |grep mongo
vagrant   3499  0.8  1.0 1003384 58692 ?       Sl   20:00   0:00 mongod --config /u01/config/mongo01/mongod.conf --oplogSize 10 --fork
```

```bash 
echo "db.version()" |mongo
MongoDB shell version v3.6.1
connecting to: mongodb://127.0.0.1:27017
MongoDB server version: 3.6.1
3.6.1
bye
```
