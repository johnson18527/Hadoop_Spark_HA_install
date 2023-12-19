# 主機配置

## master、node1、node2皆執行

### hostname設定
```
vi /etc/hosts
```

```
#hosts
master_ip  master

node1_ip  node1

node2_ip  node2

#ip與hostname之間空4格
#127.0.0.1 master、127.0.0.1 node1、127.0.0.1 node2需移除
```



### 關閉防火牆
```
systemctl stop ufw
systemctl disable ufw
systemctl status ufw
```

### 創造使用者hadoop及其資料夾
```
groupadd hadoop
useradd -r -g hadoop hadoop
mkhomedir_helper hadoop
sudo chsh  -s  /bin/bash Hadoop
chown -R hadoop.hadoop /usr/local/
chown -R hadoop.hadoop /tmp/
chown -R hadoop.hadoop /home/
```

### 創造使用者hadoop及其資料夾
```
groupadd hadoop
useradd -r -g hadoop hadoop
mkhomedir_helper hadoop
sudo chsh  -s  /bin/bash Hadoop
chown -R hadoop.hadoop /usr/local/
chown -R hadoop.hadoop /tmp/
chown -R hadoop.hadoop /home/
```

### 創造使用者hadoop及其資料夾

```
vi /etc/sudoers
```
```
#sudoers

root  ALL=(ALL:ALL) ALL

hadoop  ALL=(ALL:ALL) ALL
```
### 設定使用者hadoop密碼並切換到使用者hadoop

```
passwd hadoop
su hadoop
```
### 設定ssh
sudo權限修改
```
ssh-keygen -t rsa
cat /home/hadoop/.ssh/id_rsa.pub >> /home/hadoop/.ssh/authorized_keys
chmod 700 /home/hadoop
chmod 700 /home/hadoop/.ssh
chmod 600 /home/hadoop/.ssh/authorized_keys
chmod 600 /home/hadoop/.ssh/id_rsa
chmod 600 /home/hadoop/.ssh/id_rsa.pub
```

## master執行
### ssh連線分配
```
ssh-copy-id -i ~/.ssh/id_rsa.pub hadoop@node1
ssh-copy-id -i ~/.ssh/id_rsa.pub hadoop@node2
```
