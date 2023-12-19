#  Spark (HA)安裝
## 主機配置
|                       |master     |node1      |node1      |
|-----------------------|-----------|-----------|-----------|
|master                 |V          |V          |           |
|worker                 |V          |V          |V          |


## master、node1、node2皆執行
### 切換使用者hadoop並安裝scala
```
sudo apt-get update
sudo apt install scala
scala
#Ctrl + d 退出
```
## master執行
### 安裝Spark
```
curl -O -k https://downloads.apache.org/spark/spark-3.2.3/spark-3.2.3-bin-hadoop3.2.tgz
tar -zxvf spark-3.2.3-bin-hadoop3.2.tgz
mv spark-3.2.3-bin-hadoop3.2 spark-3.2.3
mv spark-3.2.3 /usr/local/
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
export SPARK_HOME=/usr/local/spark-3.2.3

#for RuntimeError: Python in worker has different version 3.9 than that in driver 3.8, PySpark cannot run with different minor versions. Please check environment variables PYSPARK_PYTHON and PYSPARK_DRIVER_PYTHON are correctly set.

export PYSPARK_PYTHON=/usr/bin/python3
export PYSPARK_DRIVER_PYTHON=/usr/bin/python3
export PATH=$PATH:$JAVA_HOME/bin:$ZOOKEEPER_HOME:$HADOOP_HOME/bin:$HADOOP_HOME/sbin:$SPARK_HOME/sbin
```
```
source /etc/profile
```
## master執行
### 配置Spark
配置 spark-env .sh
```
cd $SPARK_HOME/conf
cp spark-env.sh.template spark-env.sh
vi spark-env.sh
```
```
#spark-env.sh
export JAVA_HOME=/usr/local/jdk
export SPARK_HISTORY_OPTS="-Dspark.history.ui.port=4000 -Dspark.history.retainedApplications=3 -Dspark.history.fs.logDirectory=hdfs:///spark_log"
export SPARK_DAEMON_JAVA_OPTS="-Dspark.deploy.recoveryMode=ZOOKEEPER -Dspark.deploy.zookeeper.url= master:2181,node1:2181,node2:2181"
SPARK_MASTER_WEBUI_PORT=8090
SPARK_WORKER_MEMORY=1g
export SPARK_MASTER_HOST=master
```

配置worker
```
cd $SPARK_HOME/conf
cp workers.template workers
vi workers
```
```
#workers
master
node1
node2
```
配置spark-defaults.conf
```
cd $SPARK_HOME/conf
cp spark-defaults.conf.template spark-defaults.conf
vi spark-defaults.conf
```
```
#spark-defaults.conf
spark.eventLog.enabled  true
spark.eventLog.dir  hdfs:///spark_log
spark.eventLog.compress  true
#spark.master  spark://master:7077
spark.serializer  org.apache.spark.serializer.KryoSerializer
spark.driver.memory  1g
spark.executor.extraJavaOptions -XX:+PrintGCDetails -Dkey=value -Dnumbers="one two three"
spark.ui.enabled true
spark.serializer.objectStreamReset 100
spark.executor.logs.rolling.time.interval daily
spark.executor.logs.rolling.maxRetainedFiles 30
spark.ui.killEnabled true
spark.ui.liveUpdate.period 100ms
spark.ui.liveUpdate.minFlushPeriod 3s
spark.ui.port 4040
spark.history.ui.port 18080
spark.ui.retainedJobs 100
spark.ui.retainedStages 100
spark.ui.retainedTasks 1000
spark.ui.showConsoleProgress true
spark.worker.ui.retainedExecutors 100
spark.worker.ui.retainedDrivers 100
spark.sql.ui.retainedExecutions 100
spark.streaming.ui.retainedBatches 100
spark.ui.retainedDeadExecutors 100
```
### 整份Spark連同設定檔同步至node1、node2
```
scp -r /usr/local/spark-3.2.3 node1:/tmp
scp -r /usr/local/spark-3.2.3 node2:/tmp
```
## master、node1、node2皆執行
### 格式化文件移至正確路徑
```
sudo mv /tmp/spark-3.2.3 /usr/local/
```
## master執行
### 確認Hadoop是否執行
```
/home/hadoop/statusHadoop.sh
#如未執行，下指令/home/hadoop/startHadoop.sh
```
### 創造log資料夾
```
hdfs dfs -mkdir /spark_log
```
### 啟動Saprk
```
$SPARK_HOME/sbin/start-all.sh
```
### 確認Saprk是否啟動
查看瀏覽器http://master_ip:8090/
