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

# References

[1] https://docs.mongodb.com/manual/tutorial/install-mongodb-on-ubuntu/

[2] https://docs.mongodb.com/manual/reference/program/mongoexport/

[3] https://docs.mongodb.com/manual/reference/program/mongoimport/

