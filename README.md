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

# References

[1] https://docs.mongodb.com/manual/tutorial/install-mongodb-on-ubuntu/

