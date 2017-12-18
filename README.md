# MongoDB_Notes

Notes on my experience with MongoDB. Serve as a reminder just in case I forgot something (especially some syntax).

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

4. mongoimport

Import JSON file into a collection (each line is a json object). Note: truncate the current collection if any.

```bash
mongoimport --db database_name --collection collection_name --drop --file ~/downloads/primer-dataset.json
```

See [3] for original posting.

5. Sharding and Replica Set

* Sharding

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

* Replica Set

It is a group of MongoDB servers operating in a primary/secondary failover fashion. At any point there can only be one primary member within the replica set, however, you can have as many secondaries as you want.

All secondaries actively replicate data off of the current primary member so that if it fails, one of them will be able to take over quite seamlessly as the new primary.

Your application will usually only run queries against the primary member in the replica set.

![Replica Set](./Images/ReplicaSet.png)

See [4] for original posting.

# References

[1] https://docs.mongodb.com/manual/tutorial/install-mongodb-on-ubuntu/

[2] https://docs.mongodb.com/manual/reference/program/mongoexport/

[3] https://docs.mongodb.com/manual/reference/program/mongoimport/

[4] https://eladnava.com/deploy-a-highly-available-mongodb-replica-set-on-aws/

