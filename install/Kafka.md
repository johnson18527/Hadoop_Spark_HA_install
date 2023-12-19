#  Kafka安裝
## 主機配置
|                       |master     |node1      |node1      |
|-----------------------|-----------|-----------|-----------|
|Kafka                  |V          |V          |V          |


## master、node1、node2皆執行
### 切換使用者hadoop並安裝Kafka
```
su hadoop
curl -O -k  https://downloads.apache.org/kafka/3.0.2/kafka_2.13-3.0.2.tgz
tar -zxvf kafka_2.13-3.0.2.tgz
```
### 更新環境變數
```
sudo vi /etc/profile
```
```
#profile
export JAVA_HOME=/usr/local/jdk
export PATH=$PATH:$JAVA_HOME/bin
export JAVA_HOME=/usr/local/jdk
export ZOOKEEPER_HOME=/usr/local/zookeeper-3.7.1/bin/
export HADOOP_HOME=/usr/local/hadoop-3.3.1
export HADOOP_CONF_DIR=$HADOOP_HOME/etc/hadoop
export HADOOP_COMMON_HOME=$HADOOP_HOME
export HADOOP_HDFS_HOME=$HADOOP_HOME
export HADOOP_MAPRED_HOME=$HADOOP_HOME
export HADOOP_YARN_HOME=$HADOOP_HOME
export HADOOP_OPTS="-Djava.library.path=$HADOOP_HOME/lib/native"
export HADOOP_COMMON_LIB_NATIVE_DIR=$HADOOP_HOME/lib/native
export SPARK_HOME=/usr/local/spark-3.2.3
export PYSPARK_PYTHON=/usr/bin/python3
export PYSPARK_DRIVER_PYTHON=/usr/bin/python3
export KAFKA_HOME=/usr/local/kafka_2.13-3.0.2
export PATH=$PATH:$JAVA_HOME/bin:$ZOOKEEPER_HOME:$HADOOP_HOME/bin:$HADOOP_HOME/sbin:$SPARK_HOME/sbin:$KAFKA_HOME/bin
```
```
source /etc/profile
```
### 創建資料、log資料夾
```
sudo mkdir /data/kafka
sudo mkdir /data/kafka/kafkalog
cd /usr/local/kafka_2.13-3.0.2
sudo chmod 737 logs
cd /usr/local/kafka_2.13-3.0.2/logs
sudo chmod 647 kafkaServer.out
```
## master執行
### 配置Kafka node
```
sudo vi /usr/local/kafka_2.13-3.0.2/config/server.properties
```
```
#server.properties
broker.id=1
listeners=PLAINTEXT://master:9092
advertised.listeners=PLAINTEXT://master:9092
message.max.byte=5242880
default.replication.factor=2
replica.fetch.max.bytes=5242880
logs.dirs=/data/kafka/kafkalog
zookeeper.connect=master:2181,node1:2181,node2:2181
```
## node1執行
### 配置Kafka node
```
sudo vi /usr/local/kafka_2.13-3.0.2/config/server.properties
```
```
#server.properties
broker.id=2
listeners=PLAINTEXT://node1:9092
advertised.listeners=PLAINTEXT://node1:9092
message.max.byte=5242880
default.replication.factor=2
replica.fetch.max.bytes=5242880
logs.dirs=/data/kafka/kafkalog
zookeeper.connect=master:2181,node1:2181,node2:2181
```
## node2執行
### 配置Kafka node
```
sudo vi /usr/local/kafka_2.13-3.0.2/config/server.properties
```
```
#server.properties
broker.id=3
listeners=PLAINTEXT://node2:9092
advertised.listeners=PLAINTEXT://node2:9092
message.max.byte=5242880
default.replication.factor=2
replica.fetch.max.bytes=5242880
logs.dirs=/data/kafka/kafkalog
zookeeper.connect=master:2181,node1:2181,node2:2181
```
## master、node1、node2皆執行
### 建立Kafka服務
```
sudo vi /etc/systemd/system/kafka.service
```
```
[Unit]
Description=Apache Kafka Server
Documentation=http://kafka.apache.org/documentation.html
Requires=zookeeper.service
After=zookeeper.service elasticsearch.service

[Service]
Type=forking
WorkingDirectory=/usr/local/kafka_2.13-3.0.2
User=hadoop
Group=hadoop
ExecStart=/usr/local/kafka_2.13-3.0.2/bin/kafka-server-start.sh -daemon /usr/local/kafka_2.13-3.0.2/config/server.properties
ExecStop=/usr/local/kafka_2.13-3.0.2/bin/kafka-server-stop.sh /usr/local/kafka_2.13-3.0.2/config/server.properties
TimeoutSec=30
Restart=on-failure

[Install]
WantedBy=default.target
```
### 確認ZooKeeper是否啟用
```
sudo systemctl status zookeeper
#若沒有，下指令 sudo systemctl start zookeeper(每台都要開)
```
### 啟動Kafka服務
```
sudo systemctl daemon-reload
sudo systemctl start kafka
sudo systemctl enable kafka
```
### 確認Kafka服務是否啟動
```
sudo systemctl status kafka
```