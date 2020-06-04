# Kafka Multi Node Cluster Setup

## Install dependencies

```bash
$ sudo apt update
$ sudo apt install default-jdk
$ sudo apt-get install scala
# if you are true linuxoid
$ sudo apt-get install zsh
```

## ZooKeeper Cluster Setup

### Create zk User

```bash
$ sudo useradd zk -m
# use /bin/bash if you zsh is not installed
$ sudo usermod --shell /bin/zsh zk
$ sudo passwd zk
$ sudo usermod -aG sudo zk
```

### Download ZooKeeper

```bash
$ wget https://downloads.apache.org/zookeeper/zookeeper-3.6.1/apache-zookeeper-3.6.1-bin.tar.gz
$ tar -xvf apache-zookeeper-3.6.1-bin.tar.gz
$ sudo chown zk:zk -R apache-zookeeper-3.6.1-bin
$ sudo mv apache-zookeeper-3.6.1-bin /opt/zookeeper
# create folder to store data
$ sudo mkdir -p /data/zookeeper
```

### Edit ZooKeeper Configuration File

```bash
$ vim /opt/zookeeper/conf/zoo.cfg
...
# length of a tick in milliseconds
tickTime=2000
# directory used to store in-memory database
dataDir=/data/zookeeper
clientPort=2181
# maximum number of client connections
# be sure to change this value in production
# I don't know which value fits the most because out services will use zk to store data
# maxClientCnxns=0 removes the limit on concurrent connections
maxClientCnxns=0

# multi node cluster setup
initLimit=10
syncLimit=5
server.1=zookeeper1.lv:2888:3888
server.2=zookeeper2.lv:2888:3888
server.3=zookeeper3.lv:2888:3888
```

### Create Node ID file

```bash
$ sudo nano /data/zookeeper/myid
...
# enter unique id
1
```

### Setup ZooKeeper Systemd Unit Service

```bash
$ sudo vim /etc/systemd/system/zookeeper.service
```

```ini
[Unit]
Description=Zookeeper Broker [1]
Documentation=http://zookeeper.apache.org
Requires=network.target
After=network.target

[Service]
Type=forking
WorkingDirectory=/opt/zookeeper
User=zk
Group=zk
ExecStart=/opt/zookeeper/bin/zkServer.sh start /opt/zookeeper/conf/zoo.cfg
ExecStop=/opt/zookeeper/bin/zkServer.sh stop /opt/zookeeper/conf/zoo.cfg
ExecReload=/opt/zookeeper/bin/zkServer.sh restart /opt/zookeeper/conf/zoo.cfg
TimeoutSec=30
Restart=on-failure

[Install]
WantedBy=default.target
```

### Change Folder Permissions

```bash
$ sudo chown zk:zk /opt/zookeeper
$ sudo chown zk:zk /data/zookeeper
```

### Start ZooKeeper Broker

```bash
$ sudo systemctl daemon-reload
$ sudo systemctl enable zookeeper
$ sudo systemctl start zookeeper
```

---

## Kafka Cluster Setup

### Create kafka User

```bash
$ sudo useradd kafka -m
# use /bin/bash if you zsh is not installed
$ sudo usermod --shell /bin/zsh kafka
$ sudo passwd kafka
$ sudo usermod -aG sudo kafka
```

### Download Apache Kafka

```bash
$ wget https://downloads.apache.org/kafka/2.5.0/kafka_2.13-2.5.0.tgz
$ tar -xvf kafka_2.13-2.5.0.tgz
$ sudo mv kafka_2.13-2.5.0 /opt/kafka

# create folder to store data
$ sudo mkdir -p /data/kafka
```

### Edit Kafka Configuration File

```bash
$ vim /opt/kafka/config/server.properties
...
# change broker.id to a unique value on every machine
broker.id=1
# change log.dirs if you don't want to lose all data after reboot
log.dirs=/data/kafka
# ZooKeeper connection string (comma separated host:port pairs)
zookeeper.connect=kafka1.lv:2181,kafka2.lv:2181,kafka3.lv:2181
```

### Setup Kafka Systemd Unit Service

```bash
$ sudo vim /etc/systemd/system/kafka.service
```

```ini
[Unit]
Description=Apache Kafka Broker [1]
Documentation=http://kafka.apache.org/documentation.html

[Service]
Type=forking
WorkingDirectory=/opt/kafka
User=kafka
Group=kafka
ExecStart=/opt/kafka/bin/kafka-server-start.sh -daemon /opt/kafka/config/server.properties
ExecStop=/opt/kafka/bin/kafka-server-stop.sh
TimeoutSec=30
Restart=on-failure

[Install]
WantedBy=default.target
```

### Change Folder Permissions

```bash
$ sudo chown kafka:kafka /opt/kafka
$ sudo chown kafka:kafka /data/kafka
```

### Start Kafka Broker

```bash
$ sudo systemctl daemon-reload
$ sudo systemctl enable kafka
$ sudo systemctl start kafka
```

## You Are Ready 👑

![Alt Text](https://media.giphy.com/media/vFKqnCdLPNOKc/giphy.gif)
