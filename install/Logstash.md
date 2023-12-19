#  Logstash安裝
## 主機配置
|                       |master     |node1      |node1      |
|-----------------------|-----------|-----------|-----------|
|Logstash               |V          |V          |V          |


## master、node1、node2皆執行
### 切換使用者hadoop並安裝Logstash
```
su hadoop
sudo wget --no-check-certificate https://artifacts.elastic.co/downloads/logstash/logstash-7.15.0-amd64.deb
sudo dpkg -i logstash-7.15.0-amd64.deb
```
## master執行
### 配置 Logstash node(ElasticSearch與Kafak串接設定)
```
sudo vi /etc/logstash/conf.d/kafka.conf
```
```
#kafka.conf
input {
kafka {
bootstrap_servers => "master_ip:9092"
topics => ["elk-sys-log"]
}
}
output {
elasticsearch {
hosts => ["http://master_ip:9200"]
index => "es-elk-sys-log-%{+YYYY.MM.dd}"
}
}
```
## node1執行
### 配置 Logstash node(ElasticSearch與Kafak串接設定)
```
sudo vi /etc/logstash/conf.d/kafka.conf
```
```
#kafka.conf
input {
kafka {
bootstrap_servers => "node1:9092"
topics => ["elk-sys-log"]
}
}
output {
elasticsearch {
hosts => ["http://node1:9200"]
index => "es-elk-sys-log-%{+YYYY.MM.dd}"
}
}
```

## node2執行
### 配置 Logstash node(ElasticSearch與Kafak串接設定)
```
sudo vi /etc/logstash/conf.d/kafka.conf
```
```
#kafka.conf
input {
kafka {
bootstrap_servers => "node2:9092"
topics => ["elk-sys-log"]
}
}
output {
elasticsearch {
hosts => ["http://node2:9200"]
index => "es-elk-sys-log-%{+YYYY.MM.dd}"
}
}
```