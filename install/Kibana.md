#  Kibana安裝
## 主機配置
|                       |master     |node1      |node1      |
|-----------------------|-----------|-----------|-----------|
|Kibana                 |V          |           |           |


## master執行
### 切換使用者hadoop並安裝Kibana
```
su hadoop
wget --no-check-certificate https://artifacts.elastic.co/downloads/kibana/kibana-7.15.0-amd64.deb
sudo dpkg -i kibana-7.15.0-amd64.deb
```
### 配置Kibana
```
sudo vi /etc/kibana/kibana.yml
```
```
#kibana.yml
server.port: 5601
server.host: "master_ip"
server.publicBaseUrl: "http://master_ip:5601/"
elasticsearch.hosts: ["http://localhost:9200"]
xpack.security.enabled: true
elasticsearch.username: "kibana"
elasticsearch.password: "******"
```
### 啟動Kibana服務
```
sudo systemctl start kibana
sudo systemctl status kibana
sudo systemctl enable kibana
```
### 確認Kibana服務是否啟動
```
sudo systemctl status kibana
```
### 用Kibana查看ElasticSearch狀態
瀏覽器查看http://master_ip:5601
