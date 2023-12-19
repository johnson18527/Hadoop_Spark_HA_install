#  Hadoop(HA)安裝
## 主機配置
|                       |master     |node1      |node1      |
|-----------------------|-----------|-----------|-----------|
|NameNode               |V          |V          |           |
|DataNode               |V          |V          |V          |
|JournalNode            |V          |V          |V          |
|NodeManager            |V          |V          |V          |
|QuorumPeerMain         |V          |V          |V          |
|DFSZKFailoverController|V          |V          |           |
|ResourceManager        |V          |V          |           |


## master、node1、node2皆執行
### 切換使用者hadoop並安裝JDK
```
su hadoop
sudo apt-get -q update
sudo apt-get -yq install gnupg curl
curl -O -k https://cdn.azul.com/zulu/bin/zulu11.60.19-ca-jdk11.0.17-linux_x64.tar.gz
tar xvf zulu11.60.19-ca-jdk11.0.17-linux_x64.tar.gz
sudo mv zulu11.60.19-ca-jdk11.0.17-linux_x64 /usr/local
sudo mv /usr/local/zulu11.60.19-ca-jdk11.0.17-linux_x64 /usr/local/jdk
```
### 更新環境變數
```
sudo vi /etc/profile
```
```
#profile
export JAVA_HOME=/usr/local/jdk

export PATH=$PATH:$JAVA_HOME/bin
```
```
source /etc/profile
```
### 確認JAVA安裝成功
```
java -version
```
### 安裝ZooKeeper cluster
```
sudo vi /etc/ssl/openssl.cnf
```
```
#openssl.cnf
CipherString = DEFAULT:@SECLEVEL=0
```

```
curl -O -k  https://downloads.apache.org/zookeeper/zookeeper-3.7.1/apache-zookeeper-3.7.1-bin.tar.gz
tar -zxvf apache-zookeeper-3.7.1-bin.tar.gz
sudo mv apache-zookeeper-3.7.1-bin /usr/local/zookeeper-3.7.1
```
### 更新環境變數
```
sudo vi /etc/profile
```
```
#profile
export JAVA_HOME=/usr/local/jdk
export ZOOKEEPER_HOME=/usr/local/zookeeper-3.7.1/bin/
export PATH=$PATH:$JAVA_HOME/bin:$ZOOKEEPER_HOME
```
```
source /etc/profile
```
### zkdata、zkdatalog創建
```
mkdir /usr/local/zookeeper-3.7.1/zkdata

mkdir /usr/local/zookeeper-3.7.1/zkdatalog
```
## master執行
### master zk myid設定
```
echo "1" > /usr/local/zookeeper-3.7.1/zkdata/myid
```
## node1執行
### node1  zk myid設定
```
echo "2" > /usr/local/zookeeper-3.7.1/zkdata/myid
```
## node2執行
### node2  zk myid設定
```
echo "3" > /usr/local/zookeeper-3.7.1/zkdata/myid
```

## master、node1、node2皆執行
### 配置ZooKeeper
```
cd /usr/local/zookeeper-3.7.1/conf/
cp zoo_sample.cfg zoo.cfg
vi zoo.cfg
```
```
#zoo.cfg
# The number of milliseconds of each tick

tickTime=2000

# The number of ticks that the initial

# synchronization phase can take

initLimit=10

# The number of ticks that can pass between

# sending a request and getting an acknowledgement

syncLimit=5

# the directory where the snapshot is stored.

# do not use /tmp for storage, /tmp here is just

# example sakes.

dataDir=/usr/local/zookeeper-3.7.1/zkdata

dataLogDir=/usr/local/zookeeper-3.7.1/zkdatalog

# the port at which the clients will connect

clientPort=2181

server.1=master:2888:3888

server.2=node1:2888:3888

server.3=node2:2888:3888
```
### 設定zookeeper服務
```
sudo vi /etc/systemd/system/zookeeper.service
```

```
#zookeeper.service
[Unit]
Description=Zookeeper Daemon
Documentation=http://zookeeper.apache.org
Requires=network.target
After=network.target
[Service]
Type=forking
WorkingDirectory=/usr/local/zookeeper-3.7.1
User=hadoop
Group=hadoop
ExecStart=/usr/local/zookeeper-3.7.1/bin/zkServer.sh start /usr/local/zookeeper-3.7.1/conf/zoo.cfg
ExecStop=/usr/local/zookeeper-3.7.1/bin/zkServer.sh stop /usr/local/zookeeper-3.7.1/conf/zoo.cfg
ExecReload=/usr/local/zookeeper-3.7.1/bin/zkServer.sh restart /usr/local/zookeeper-3.7.1/conf/zoo.cfg
TimeoutSec=30
Restart=on-failure

[Install]
WantedBy=default.target

```
### 啟動zookeeper服務
```
sudo systemctl daemon-reload
sudo systemctl start zookeeper
sudo systemctl enable zookeeper
```
### 確認zookeeper服務
```
sudo systemctl status zookeeper
```

## master執行
### 安裝Hadoop
```
curl -O -k https://dlcdn.apache.org/hadoop/common/hadoop-3.3.1/hadoop-3.3.1.tar.gz
tar -zxvf hadoop-3.3.1.tar.gz
sudo mv hadoop-3.3.1 /usr/local/
hadoop version
```
## master、node1、node2皆執行
### 更新環境變數
```
sudo vi /etc/profile
```
```
#profile
export JAVA_HOME=/usr/local/jdk
export CLASS_PATH=.:$JAVA_HOME/lib
export ZOOKEEPER_HOME=/usr/local/zookeeper-3.7.1/bin/
export HADOOP_HOME=/usr/local/hadoop-3.3.1
export HADOOP_CONF_DIR=$HADOOP_HOME/etc/hadoop
export HADOOP_COMMON_HOME=$HADOOP_HOME
export HADOOP_HDFS_HOME=$HADOOP_HOME
export HADOOP_MAPRED_HOME=$HADOOP_HOME
export HADOOP_YARN_HOME=$HADOOP_HOME
export HADOOP_OPTS="-Djava.library.path=$HADOOP_HOME/lib/native"
export HADOOP_COMMON_LIB_NATIVE_DIR=$HADOOP_HOME/lib/native
export PATH=$PATH:$JAVA_HOME/bin:$ZOOKEEPER_HOME:$HADOOP_HOME/bin:$HADOOP_HOME/sbin
```
```
source /etc/profile
```
## master執行
### 配置Hadoop
配置 hadoop-env
```
cd $HADOOP_CONF_DIR
sudo vi hadoop-env.sh
```
```
#hadoop-env.sh
export JAVA_HOME=/usr/local/jdk
export HADOOP_PID_DIR=/usr/local/hadoop-3.3.1/pid
```
配置 core-site.xml
```
cd $HADOOP_CONF_DIR
sudo vi core-site.xml
```
```
#core-site.xml
<configuration>
<property>
<name>fs.defaultFS</name>
<value>hdfs://ns</value>
</property>
<property>
<name>hadoop.tmp.dir</name>
<value>/usr/local/hadoop-3.3.1/tmp</value>
</property>
<property>
<name>ha.zookeeper.quorum</name>
<value>master:2181,node1:2181,node2:2181</value>
<name>hadoop.http.staticuser.user</name>
<value>hadoop</value>
</property>
</configuration>
```
配置 hdfs-site.xml
```
cd $HADOOP_CONF_DIR
sudo vi hdfs-site.xml
```
```
#hdfs-site.xml
<configuration>
<property>
<name>dfs.nameservices</name>
<value>ns</value>
</property>
<property>
<name>dfs.ha.namenodes.ns</name>
<value>nn1,nn2</value>
</property>
<property>
<name>dfs.namenode.rpc-address.ns.nn1</name>
<value>master:9000</value>
</property>
<property>
<name>dfs.namenode.http-address.ns.nn1</name>
<value>master:9870</value>
</property>
<property>
<name>dfs.namenode.rpc-address.ns.nn2</name>
<value>node1:9000</value>
</property>
<property>
<name>dfs.namenode.http-address.ns.nn2</name>
<value>node1:9870</value>
</property>
<property>
<name>dfs.namenode.shared.edits.dir</name>
<value>qjournal://master:8485;node1:8485;/ns</value>
</property>
<property>
<name>dfs.journalnode.edits.dir</name>
<value>/usr/local/hadoop-3.3.1/journaldata</value>
</property>
<property>
<name>dfs.ha.automatic-failover.enabled</name>
<value>true</value>
</property>
<property>
<name>dfs.client.failover.proxy.provider.ns</name>
<value>org.apache.hadoop.hdfs.server.namenode.ha.ConfiguredFailoverProxyProvider</value>
</property>
<property>
<name>dfs.ha.fencing.methods</name>
<value>
sshfence
shell(/bin/true)
</value>
</property>
<property>
<name>dfs.ha.fencing.ssh.private-key-files</name>
<value>/home/hadoop/.ssh/id_rsa</value>
</property>
<property>
<name>dfs.ha.fencing.ssh.connect-timeout</name>
<value>30000</value>
</property>
<property>
<name>dfs.namenode.heartbeat.recheck-interval</name>
<value>3000</value>
</property>
<property>
<name>dfs.heartbeat.interval</name>
<value>2</value>
</property>
<property>
<name>dfs.blockreport.intervalMsec</name>
<value>21600000</value>
</property>
<property>
<name>dfs.replication</name>
<value>3</value>
</property>
<property>
<name>dfs.namenode.replication.min</name>
<value>1</value>
</property>
<property>
<name>io.bytes.per.checksum</name>
<value>512</value>
</property>

</configuration>
```
配置mapred-site.xml
```
cd $HADOOP_CONF_DIR
sudo vi mapred-site.xml
```
```
#mapred-site.xml
<configuration>
<property>
<name>mapreduce.framework.name</name>
<value>yarn</value>
</property>
<property>
<name>yarn.app.mapreduce.am.env</name>
<value>HADOOP_MAPRED_HOME=${HADOOP_HOME}</value>
</property>
<property>
<name>mapreduce.map.env</name>
<value>HADOOP_MAPRED_HOME=${HADOOP_HOME}</value>
</property>
<property>
<name>mapreduce.reduce.env</name>
<value>HADOOP_MAPRED_HOME=${HADOOP_HOME}</value>
</property>
<property>
<name>mapreduce.task.timeout</name>
<value>600000</value>
</property>
<property>
<name>mapreduce.map.maxattempts</name>
<value>4</value>
</property>
<property>
<name>mapreduce.reduce.maxattempts</name>
<value>4</value>
</property>
<property>
<name>mapreduce.am.max-attempts</name>
<value>2</value>
</property>
<property>
<name>mapreduce.job.maxtaskfailures.per.tracker</name>
<value>3</value>
</property>
</configuration>
```

配置yarn-site.xml
```
cd $HADOOP_CONF_DIR
sudo vi yarn-site.xml
```
```
#yarn-site.xml
<configuration>
<property>
<name>yarn.resourcemanager.ha.enabled</name>
<value>true</value>
</property>
<property>
<name>yarn.resourcemanager.cluster-id</name>
<value>yrc</value>
</property>
<property>
<name>yarn.resourcemanager.ha.rm-ids</name>
<value>rm1,rm2</value>
</property>
<property>
<name>yarn.resourcemanager.zk-address</name>
<value>master:2181,node1:2181,node2:2181</value>
</property>
<property>
<name>yarn.resourcemanager.aux-services</name>
<value>mapreduce_shuffle</value>
</property>
<property>
<name>yarn.nodemanager.aux-services</name>
<value>mapreduce_shuffle</value>
</property>
<property>
<name>yarn.nodemanager.auxservices.mapreduce.shuffle.class</name>
<value>org.apache.hadoop.mapred.ShuffleHandler</value>
</property>
<property>
<name>yarn.resourcemanager.hostname.rm1</name>
<value>master</value>
</property>
<property>
<name>yarn.resourcemanager.address.rm1</name>
<value>master:8032</value>
</property>
<property>
<name>yarn.resourcemanager.scheduler.address.rm1</name>
<value>master:8030</value>
</property>
<property>
<name>yarn.resourcemanager.webapp.address.rm1</name>
<value>master:8088</value>
</property>
<property>
<name>yarn.resourcemanager.resource-tracker.address.rm1</name>
<value>master:8031</value>
</property>
<property>
<name>yarn.resourcemanager.admin.address.rm1</name>
<value>master:8033</value>
</property>
<property>
<name>yarn.resourcemanager.ha.admin.address.rm1</name>
<value>master:23142</value>
</property>
<property>
<name>yarn.resourcemanager.hostname.rm2</name>
<value>node1</value>
</property>
<property>
<name>yarn.resourcemanager.address.rm2</name>
<value>node1:8032</value>
</property>
<property>
<name>yarn.resourcemanager.scheduler.address.rm2</name>
<value>node1:8030</value>
</property>
<property>
<name>yarn.resourcemanager.webapp.address.rm2</name>
<value>node1:8088</value>
</property>
<property>
<name>yarn.resourcemanager.resource-tracker.address.rm2</name>
<value>node1:8031</value>
</property>
<property>
<name>yarn.resourcemanager.admin.address.rm2</name>
<value>node1:8033</value>
</property>
<property>
<name>yarn.resourcemanager.ha.admin.address.rm2</name>
<value>node1:23142</value>
</property>
<property>
<name>yarn.resourcemanager.am.max-attempts</name>
<value>2</value>
</property>
<property>
<name>yarn.resourcemanager.nm.liveness-monitor.expiry-interval-ms</name>
<value>600000</value>
</property>
<property>
<name>yarn.resourcemanager.store.class</name>
<value>org.apache.hadoop.yarn.server.resourcemanager.recovery.FileSystemRMStateStore</value>
</property>
</configuration>
```

配置workers
```
cd $HADOOP_CONF_DIR
sudo vi workers
```
```
#workers
master
node1
node2
```
### 整份Hadoop連同設定檔同步至node1、node2
```
scp -r /usr/local/hadoop-3.3.1 node1:/home/node1
scp -r /usr/local/hadoop-3.3.1 node2:/home/node2
```

## node1、node2執行
### Hadoop移到正確路徑
```
sudo mv hadoop-3.3.1 /usr/local/
```
## master、node1、node2皆執行
### 確認ZooKeeper狀態
```
zkServer.sh status
#確認所有機器都有執行zookeeper，如果沒有，下指令zkServer.sh start
```
### 格式化資料夾
```
/usr/local/hadoop-3.3.1/sbin/hadoop-daemon.sh start journalnode
sudo chmod -w /usr/local/hadoop-3.3.1/
sudo chmod -w /usr/local/hadoop-3.3.1/pid
sudo chmod -w /usr/local/hadoop-3.3.1/logs
sudo chmod -w /usr/local/hadoop-3.3.1/pid/*
sudo chmod -w /usr/local/hadoop-3.3.1/logs/*
```
## master執行
### 格式化文件並同步至node1、node2

```
hdfs namenode -format ns
scp -r /usr/local/hadoop-3.3.1/tmp node1:/tmp
scp -r /usr/local/hadoop-3.3.1/tmp node2:/tmp
```
## master、node1、node2皆執行
### 格式化文件移至正確路徑
```
sudo mv /tmp/tmp/ /usr/local/hadoop-3.3.1/
```
## master執行
### 啟動Hadoop
```
hdfs zkfc –formatZK
start-dfs.sh
start-yarn.sh
hdfs haadmin -transitionToStandby --forcemanual nn2
hdfs haadmin -transitionToStandby --forcemanual nn3
hdfs haadmin -transitionToActive --forcemanual nn1
```
### 確認Hadoop是否成功開啟
用瀏覽器查看http://master_ip:9870
### 相關腳本
啟動腳本
```
vi /home/hadoop/startHadoop.sh
```
```
#startHadoop.sh
#!/usr/bin/env bash
hadoop-daemon.sh start journalnode
start-dfs.sh
start-yarn.sh
```
```
chmod +x /home/hadoop/startHadoop.sh
```
關閉腳本
```
vi /home/hadoop/startHadoop.sh
```
```
#startHadoop.sh
#!/usr/bin/env bash
stop-dfs.sh
stop-yarn.sh
```
```
chmod +x /home/hadoop/stopHadoop.sh
```
狀態確認腳本
```
vi /home/hadoop/statusHadoop.sh
```
```
#statusHadoop.sh
#!/usr/bin/env bash
jps
```
```
chmod +x /home/k8s/statusHadoop.sh
```
### 腳本同步至node1、node2
```
scp /home/hadoop/startHadoop.sh node1:/home/hadoop
scp /home/hadoop/stopHadoop.sh node1:/home/hadoop
scp /home/hadoop/statusHadoop.sh node1:/home/hadoop
scp /home/hadoop/startHadoop.sh node2:/home/hadoop
scp /home/hadoop/stopHadoop.sh node2:/home/hadoop
scp /home/hadoop/statusHadoop.sh node2:/home/hadoop
```