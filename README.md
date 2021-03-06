# mongo-cluster-setup

# Version: 4.0.4

# Suppose we have aws instance as following:
### 10.0.40.123
### 10.0.40.195
### 10.0.40.252

# How do we group a replica set

### first memeber: (10.0.40.123)
### $sudo vim /etc/mongod.conf
### $sudo service mongod start
### $mongo

### in mongo shell:
### cfg = { _id: "rs0","protocolVersion" : NumberLong(1), members: [{_id: 0,host: "10.0.40.123:27017"},{_id: 1,host: "10.0.40.195:27017"},{_id: 2,host: "10.0.40.252:27017"}]}

### cfg.members[0].priority = 1
### cfg.members[1].priority = 0.5
### cfg.members[2].priority = 0.5

### rs.conf(cfg);
### rs.initiate(cfg);

### second member: (10.0.40.195)
### $sudo vim /etc/mongod.conf
### $sudo service mongod start
### $mongo
### cfg = { _id: "rs0","protocolVersion" : NumberLong(1), members: [{_id: 0,host: "10.0.40.123:27017"},{_id: 1,host: "10.0.40.195:27017"},{_id: 2,host: "10.0.40.252:27017"}]}

### cfg.members[0].priority = 1
### cfg.members[1].priority = 0.5
### cfg.members[2].priority = 0.5

### rs.conf(cfg);
### rs.initiate(cfg);
### rs.slaveOk()

### third member: (10.0.40.252)
### $sudo vim /etc/mongod.conf
### $sudo service mongod start
### $mongo
### cfg = { _id: "rs0","protocolVersion" : NumberLong(1), members: [{_id: 0,host: "10.0.40.123:27017"},{_id: 1,host: "10.0.40.195:27017"},{_id: 2,host: "10.0.40.252:27017"}]}

### cfg.members[0].priority = 1
### cfg.members[1].priority = 0.5
### cfg.members[2].priority = 0.5

### rs.conf(cfg);
### rs.initiate(cfg);
### rs.slaveOk()

# spring boot config

### As mongo official doc said:
### https://docs.mongodb.com/manual/reference/connection-string/
### mongodb://mongodb0.example.com:27017,mongodb1.example.com:27017,mongodb2.example.com:27017/admin?replicaSet=myRepl

### and we have our spring boot config file "application.yml" as below:
### spring.data.mongodb.uri=mongodb://10.0.40.123:27017,10.0.40.195:27017,10.0.40.252:27017/dbName?replicaSet=rs0

# If you face following trouble:

1. 
### rs0:SECONDARY> show dbs
> 2019-09-27T07:20:46.394+0000 E QUERY    [js] Error: listDatabases failed:{
>        "operationTime" : Timestamp(1569568839, 1),
>        "ok" : 0,
>        "errmsg" : "not master and slaveOk=false",
>        "code" : 13435,
>        "codeName" : "NotMasterNoSlaveOk",
>        "$clusterTime" : {
>                "clusterTime" : Timestamp(1569568839, 1),
>                "signature" : {
>                        "hash" : BinData(0,"AAAAAAAAAAAAAAAAAAAAAAAAAAA="),
>                        "keyId" : NumberLong(0)
>                }
>        }
> } :

### solution:

> rs0:SECONDARY> rs.slaveOk()
> rs0:SECONDARY> show dbs
> admin       0.000GB
> config      0.000GB
> local       0.088GB

2. 
### Failed to unlink socket file /tmp/mongodb-27017.sock Unknown error

solution:
$rm /tmp/mongodb-27017.sock
###########################
InvalidReplicaSetConfig: Our replica set config is invalid or we are not a member of it
Sometimes, kubernetes cluster restart. And comes t

solution:
rs.reconfig(cfg,{force:true})

3. ### stuck in RECOVERING state (Mongodb replica set status showing “RECOVERING”
)
1. Login to RECOVERING instance
2. Delete data from existing db which will be /data/db
3. Restart this RECOVERING instance

Reference:
# https://docs.mongodb.com/manual/tutorial/deploy-replica-set-for-testing/
# https://docs.mongodb.com/manual/release-notes/4.0-upgrade-replica-set/
# https://docs.bitnami.com/vmware-templates/infrastructure/mongodb/get-started/understand-cluster-config/
# https://rammusxu.github.io/2019/05/24/InvalidReplicaSetConfig-Our-replica-set-config-is-invalid-or-we-are-not-a-member-of-it/
# https://stackoverflow.com/questions/14332010/mongodb-replica-set-status-showing-recovering
