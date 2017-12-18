# MongoDB Notes

Notes on my experience with MongoDB. Serve as a reminder just in case I forgot something (especially some syntax).

#### Table of Contents

[1. Install MongoDB 3.6 on Ubuntu 16 LTS](#tip1)  
[2. Start MongoDB as a service](#tip2)  
[3. mongoexport](#tip3)  
[4. mongoimport](#tip4)  
[5. Sharding](#tip5)  
[6. Replica Set](#tip6)  
[7. Production-leveled setup for MongoDB](#tip7) 
[8. Enable sharding for a collection](#tip8) 

<a name="tip1"></a>
1. Install MongoDB 3.6 on Ubuntu 16 LTS

The last echo commands are for fixing the version to 3.6 (no automatic update whenever typing "sudo apt-get update").

```bash
sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv 2930ADAE8CAF5059EE73BB4B58712A2291FA4AD5
echo "deb [ arch=amd64,arm64 ] https://repo.mongodb.org/apt/ubuntu xenial/mongodb-org/3.6 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-3.6.list
sudo apt-get update
sudo apt-get install -y mongodb-org
echo "mongodb-org hold" | sudo dpkg --set-selections
echo "mongodb-org-server hold" | sudo dpkg --set-selections
echo "mongodb-org-shell hold" | sudo dpkg --set-selections
echo "mongodb-org-mongos hold" | sudo dpkg --set-selections
echo "mongodb-org-tools hold" | sudo dpkg --set-selections
```

Allow MongoDB to be connected by any machine:

```bash
nano /etc/mongod.conf
```

```ini
# Comment out this line below.
# bind_ip = 127.0.0.1
```

See [1] for the original posting.

<a name="tip2"></a>
2. Start MongoDB as a service

By default, MongoDB is not automatically started when then system boots.

```bash
sudo nano /etc/systemd/system/mongodb.service
```

Fill in the content:

```ini
[Unit]
Description=High-performance, schema-free document-oriented database
After=network.target

[Service]
User=mongodb
ExecStart=/usr/bin/mongod --quiet --config /etc/mongod.conf

[Install]
WantedBy=multi-user.target
```

Then enable automatic starting:

```bash
sudo systemctl enable mongodb
```

Better to restart the machine! Lastly, check the status:

```bash
sudo systemctl status mongodb
```

<a name="tip3"></a>
3. mongoexport

Export a collection to a JSON file (each line is a json object).

```bash
mongoexport --host localhost --db database_name --collection collection_name --out targetproduction.json
```

Export a collection to a CSV file (we must indicate which fields in each document to export).

```bash
mongoexport --host localhost --db database_name --collection collection_name --type=csv /fields:a_specific_field,another_field --out D:\mydata.csv
```

See [2] for original posting.

<a name="tip4"></a>
4. mongoimport

Import JSON file into a collection (each line is a json object). Note: truncate the current collection if any.

```bash
mongoimport --db database_name --collection collection_name --drop --file ~/downloads/primer-dataset.json
```

See [3] for original posting.

<a name="tip5"></a>
5. Sharding

MongoDB supports horizontal scaling through sharding. Sharding is a method for distributing data across multiple machines.

Each shard contains a subset or a replica of a dataset.

To distribute the documents in a collection, MongoDB partitions the collection using the shard key. The shard key consists of an immutable field or fields that exist in every document in the target collection.

A sharded collection can have only one shard key. You cannot change the shard key after sharding, nor can you unshard a sharded collection.

If queries do not include the shard key or the prefix of a compound shard key, mongos performs a broadcast operation, querying all shards in the sharded cluster.

A database can have a mixture of sharded and unsharded collections.

Unsharded collections are stored on a primary shard. Each database has its own primary shard.

Sharding Strategy:

   - Hashed
   
   While a range of shard keys may be “close”, their hashed values are unlikely to be on the same chunk. Data distribution based on hashed values facilitates more even data distribution.
   
   However, hashed distribution means that ranged-based queries on the shard key are less likely to target a single shard, resulting in more cluster wide broadcast operations.

   ![Hashed Sharding](./Images/HashSharding.png)
   
   - Ranged
   
   A range of shard keys whose values are "close" are more likely to reside on the same chunk.
   
   Poorly considered shard keys can result in uneven distribution of data, which can negate some benefits of sharding or can cause performance bottlenecks.
   
   ![Ranged Sharding](./Images/RangedSharding.png)

<a name="tip6"></a>
6. Replica Set

It is a group of MongoDB servers operating in a primary/secondary failover fashion. At any point there can only be one primary member within the replica set, however, you can have as many secondaries as you want.

All secondaries actively replicate data off of the current primary member so that if it fails, one of them will be able to take over quite seamlessly as the new primary.

Your application will usually only run queries against the primary member in the replica set.

![Replica Set](./Images/ReplicaSet.png)

See [4] for original posting.

<a name="tip7"></a>
7. Production-leveled setup for MongoDB

A big picture.

![Production Setup](./Images/ProductionSetup2.png)

* Configure Config Server

Note: They themselves should be in a Replica Set, too!

```bash
nano /etc/mongod.conf
```

```ini
replication:
  replSetName: configReplSet

sharding:
  clusterRole: configsvr
  
net:
   bindIp: 0.0.0.0
```

Then restart the service:

```bash
sudo service mongod restart
```

Then, connect to any of the Config Servers:

```javascript
rs.initiate( { _id: "configReplSet", configsvr: true, members: [ { _id: 0, host: "mongoconfig01:27019" }, { _id: 1, host: "mongoconfig02:27019" }, { _id: 2, host: "mongoconfig03:27019" } ] } )

rs.status()
```

See [6] for why port 27019 is used by default.

* Configure Shards

Note: Each shard is also a Replica Set!

```bash
nano /etc/mongod.conf
```

```ini
sharding:
   clusterRole: shardsvr
replication:
   replSetName: rs0
net:
   bindIp: 0.0.0.0
```

Then restart the service:

```bash
sudo service mongod restart
```

Connect to one of the MongoDB instances to initialize the replica set and declare its members.

```javascript
rs.initiate(
  {
    _id : rs0,
    members: [
      { _id : 0, host : "s1-mongo1.example.net:27018" },
      { _id : 1, host : "s1-mongo2.example.net:27018" },
      { _id : 2, host : "s1-mongo3.example.net:27018" }
    ]
  }
)
rs.status()
```

See [6] for why port 27018 is used by default.

* Configure MongoDB Router

```bash
nano /etc/mongos.conf
```

```ini
# Remove storage section from the config file!
sharding:
  configDB:configReplSet/mongoconfig01:27019,mongoconfig02:27019,mongoconfig03:27019
```

Then restart the service:

```bash
sudo service mongos restart
```

Now connecting the dots! First, connect to the MongoDB Router (mongos):

Note: mongod and mongos use the same port by default. See [6].

And run these commands:

```javascript
sh.addShard("rs0/mongosh01db01:27018")
sh.addShard("rs1/mongosh02db01:27018")

sh.status()
```

Now, within our application, connect to Mongodb Router (mongos) and query as usual.

See [5] for original posting.

<a name="tip8"></a>
8. Enable sharding for a collection

Before you can shard a collection, you must enable sharding for the collection’s database. Enabling sharding for a database does not redistribute data but make it possible to shard the collections in that database.

Once you enable sharding for a database, MongoDB assigns a primary shard for that database where MongoDB stores all data in that database.

First, connect to mongos (MongoDB Router). Then execute the commands:

```bash
sh.enableSharding("mydatabase")
sh.shardCollection("mydatabase.mycollection", { { "myshardkey" : "hashed" } } )
```

MongoDB mongos instances route queries and write operations to shards in a sharded cluster. mongos provide the only interface to a sharded cluster from the perspective of applications. Applications *never* connect or communicate directly with the shards.

The mongos tracks what data is on which shard by caching the metadata from the config servers. The mongos uses the metadata to route operations from applications and clients to the mongod instances. A mongos has no persistent state and consumes minimal system resources.

The most common practice is to run mongos instances on the same systems as your application servers, but you can maintain mongos instances on the shards or on other dedicated resources.

The mongos merges the data from each of the targeted shards and returns the result document. 

mongos performs a broadcast operation for queries that do not include the shard key, routing queries to all shards in the cluster.

See [7], [8] for original posting.

# References

[1] https://docs.mongodb.com/manual/tutorial/install-mongodb-on-ubuntu/

[2] https://docs.mongodb.com/manual/reference/program/mongoexport/

[3] https://docs.mongodb.com/manual/reference/program/mongoimport/

[4] https://eladnava.com/deploy-a-highly-available-mongodb-replica-set-on-aws/

[5] http://codingmiles.com/mongodb-sharded-cluster-deployment/

[6] https://docs.mongodb.com/manual/reference/default-mongodb-port/

[7] https://docs.mongodb.com/manual/tutorial/deploy-shard-cluster/

[8] https://docs.mongodb.com/v3.4/core/sharded-cluster-query-router/

