# MongoDB Administration

This is MongoDB Administration related guide of how to create **Replica Sets, Shards, etc.**

# Creating shard

## Creating Replica Sets

Shards consists of one or multiple replica sets, CSRS (Config Server Replica Set) and **mongos** instance

Replica set is the **mongod** instances, CSRS is also **mongod** but with the special configuration option, **mongos** is used as interface for shards.

So in order to create shard we need to: first create replica set, create csrs, create and configure sharded cluster

### Replica set configuration files

Below are 3 config files for a first replica set

#### node1.conf

```
sharding:
  clusterRole: shardsvr
storage:
  wiredTiger:
     engineConfig:
        cacheSizeGB: .1

net:
   bindIp: 192.168.103.100,localhost
   port: 27001

storage:
   dbPath: /var/mongodb/db/1

processManagement:
   fork: true

systemLog:
   destination: file
   path: /var/mongodb/db/1/mongod.log
   logAppend: true

replication:
   replSetName: m103-repl

security:
   authorization: enabled
   keyFile: /var/mongodb/pki/m103-keyfile

```

#### node2.conf

```
sharding:
  clusterRole: shardsvr
storage:
  wiredTiger:
     engineConfig:
        cacheSizeGB: .1

net:
   bindIp: 192.168.103.100,localhost
   port: 27002

storage:
   dbPath: /var/mongodb/db/2

processManagement:
   fork: true

systemLog:
   destination: file
   path: /var/mongodb/db/2/mongod.log
   logAppend: true

replication:
   replSetName: m103-repl

security:
   authorization: enabled
   keyFile: /var/mongodb/pki/m103-keyfile


```

#### node3.conf

```
sharding:
  clusterRole: shardsvr
storage:
  wiredTiger:
     engineConfig:
        cacheSizeGB: .1

net:
   bindIp: 192.168.103.100,localhost
   port: 27003

storage:
   dbPath: /var/mongodb/db/3

processManagement:
   fork: true

systemLog:
   destination: file
   path: /var/mongodb/db/3/mongod.log
   logAppend: true

replication:
   replSetName: m103-repl

security:
   authorization: enabled
   keyFile: /var/mongodb/pki/m103-keyfile


```

Be cautious of copy pasting and changing config files. You can mess around with **db** or **log** pathes and mongod will not give you any clue what is wrong.

If you accidentally runned mongod instances with the same log path, all data are entangled and you need to delete log path folder

Here are another replica set config files

#### node4.conf

```
storage:
  dbPath: /var/mongodb/db/4
  wiredTiger:
     engineConfig:
        cacheSizeGB: .1
net:
  bindIp: 192.168.103.100,localhost
  port: 27004
security:
  keyFile: /var/mongodb/pki/m103-keyfile
systemLog:
  destination: file
  path: /var/mongodb/db/4/mongod.log
  logAppend: true
processManagement:
  fork: true
operationProfiling:
  slowOpThresholdMs: 50
replication:
  replSetName: m103-repl-2
sharding:
  clusterRole: shardsvr
```

#### node5.conf

```
storage:
  dbPath: /var/mongodb/db/5
  wiredTiger:
     engineConfig:
        cacheSizeGB: .1
net:
  bindIp: 192.168.103.100,localhost
  port: 27005
security:
  keyFile: /var/mongodb/pki/m103-keyfile
systemLog:
  destination: file
  path: /var/mongodb/db/5/mongod.log
  logAppend: true
processManagement:
  fork: true
operationProfiling:
  slowOpThresholdMs: 50
replication:
  replSetName: m103-repl-2
sharding:
  clusterRole: shardsvr
```

#### node6.conf

```
storage:
  dbPath: /var/mongodb/db/6
  wiredTiger:
     engineConfig:
        cacheSizeGB: .1
net:
  bindIp: 192.168.103.100,localhost
  port: 27006
security:
  keyFile: /var/mongodb/pki/m103-keyfile
systemLog:
  destination: file
  path: /var/mongodb/db/6/mongod.log
  logAppend: true
processManagement:
  fork: true
operationProfiling:
  slowOpThresholdMs: 50
replication:
  replSetName: m103-repl-2
sharding:
  clusterRole: shardsvr
```

### Running replica set

You should run replica set with `mongod -f node1.conf` command for first node and similar commands for other 2

Some things should be already done though, like:

- `dbPath` already should be created

```
mkdir /var/mongodb/db/{1,2,3}
```

- keyfile should exist

```
sudo mkdir -p /var/mongodb/pki
sudo chown vagrant:vagrant -R /var/mongodb
openssl rand -base64 741 > /var/mongodb/pki/m103-keyfile
chmod 600 /var/mongodb/pki/m103-keyfile
```

Once you runned `mongod` command, you can check if `mongod` instances really are created with `pidof mongod` command. If necessary you can kill process by pid with `kill <pid>` or `killall mongod` command


### Initiating replica set

Before creating replica set user, we need to initiate with `rs.initiate()`

After that you can check RS status with `rs.status()`. After some time you will get `PRIMARY` postfix in mongo shell

### Creating user

Before adding more `mongod` instances to RS if no user was created before we need to create `root` user.

Connect with `mongod --port <port number>`

Switch to `admin` database with `use admin`

And create root user

```
db.createUser({
  user: "m103-admin",
  pwd: "m103-pass",
  roles : [ "root" ]
})
```

### Adding RS members

We need to reconnect with `mongo --host "m103-repl/192.168.103.100:27001" -u "m103-admin" -p "m103-pass" --authenticationDatabase "admin"`

And then add more members with 
```
rs.add("m103:27002")
rs.add("m103:27003")
```

### Create second RS

You need to also initiate second RS the same way