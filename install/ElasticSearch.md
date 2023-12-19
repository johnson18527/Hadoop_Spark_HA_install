#  ElasticSearch安裝
## 主機配置
|                       |master     |node1      |node1      |
|-----------------------|-----------|-----------|-----------|
|ElasticSearch          |V          |V          |V          |


## master、node1、node2皆執行
### 切換使用者hadoop並安裝ElasticSearch
```
su hadoop
wget --no-check-certificate  https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-7.15.0-amd64.deb
wget --no-check-certificate  https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-7.15.0-amd64.deb.sha512
shasum -a 512 -c elasticsearch-7.15.0-amd64.deb.sha512
sudo dpkg -i elasticsearch-7.15.0-amd64.deb
sudo mkdir /data
sudo mkdir /data/elasticsearch
sudo chown elasticsearch:elasticsearch /data/ -R
```
### 配置jvm.options
```
sudo vi /etc/elasticsearch/jvm.options
```
```
#jvm.options
# 需新增
-Xms1g
-Xmx1g
```
## master執行
### 配置 es-cluster
```
sudo vi /etc/elasticsearch/elasticsearch.yml
```
```
#elasticsearch.yml
cluster.name: es-cluster
node.name:  es-master
path.data: /data/elasticsearch
path.logs: /data/elasticsearch
network.host: 0.0.0.0
http.port: 9200
discovery.zen.ping.unicast.hosts: ["master_ip:9300", "node1_ip.3:9300", "node2_ip.3:9300"]
cluster.initial_master_nodes: ["es-master"]
node.roles: ["master","data"]
network.publish_host: master_ip
transport.port: 9300
http.cors.enabled: true
http.cors.allow-origin: "*"
xpack.security.enabled: false
```
## node1執行
### 配置 es-cluster
```
sudo vi /etc/elasticsearch/elasticsearch.yml
```
```
#elasticsearch.yml
cluster.name: es-cluster
node.name:  es-node1
path.data: /data/elasticsearch
path.logs: /data/elasticsearch
network.host: 0.0.0.0
http.port: 9200
discovery.zen.ping.unicast.hosts: ["master_ip:9300", "node1_ip.3:9300", "node2_ip.3:9300"]
cluster.initial_master_nodes: ["es-master"]
node.roles: ["master","data"]
network.publish_host: node1_ip
transport.port: 9300
http.cors.enabled: true
http.cors.allow-origin: "*"
xpack.security.enabled: false
```
## node2執行
### 配置 es-cluster
```
sudo vi /etc/elasticsearch/elasticsearch.yml
```
```
#elasticsearch.yml
cluster.name: es-cluster
node.name:  es-node2
path.data: /data/elasticsearch
path.logs: /data/elasticsearch
network.host: 0.0.0.0
http.port: 9200
discovery.zen.ping.unicast.hosts: ["master_ip:9300", "node1_ip.3:9300", "node2_ip.3:9300"]
cluster.initial_master_nodes: ["es-master"]
node.roles: ["master","data"]
network.publish_host: node2_ip
transport.port: 9300
http.cors.enabled: true
http.cors.allow-origin: "*"
xpack.security.enabled: false
```
## master、node1、node2皆執行
### 啟動ElasticSearch服務
```
sudo systemctl daemon-reload
sudo systemctl enable elasticsearch
sudo systemctl restart elasticsearch
```
### 確認ElasticSearch服務是否啟動
```
sudo systemctl status elasticsearch
```
## master執行
### ElasticSearch狀態監測
```
sudo apt update
sudo apt-get remove docker docker-engine docker.io
sudo apt install docker.io
docker –version
sudo systemctl start docker
sudo systemctl enable docker
sudo systemctl status docker
echo -n | openssl s_client -showcerts -connect registry-1.docker.io:443 2>/dev/null | sed -ne '/-BEGIN CERTIFICATE-/,/-END CERTIFICATE-/p' >> /etc/ssl/certs/ca-certificates.crt
service docker restart
docker pull mobz/elasticsearch-head:5
docker run -d -p 9100:9100 docker.io/mobz/elasticsearch-head:5
```
### 查看ElasticSearch狀態
瀏覽器查看http://master_ip:9100/