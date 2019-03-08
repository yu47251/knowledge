# Zookeeper

## Zookeeper 安装

### 解压
```
sudo tar -zxvf zookeeper-3.4.8.tar.gz
```
### 目录重命名
```
sudo mv zookeeper-3.4.8 zookeeper
```
### 加组和用户
```
groupadd zookeeper
useradd -g zookeeper zookeeper
```
### 修改用户组和用户
```
sudo chown -R zookeeper:zookeeper zookeeper/
```
### 修改环境变量
编辑 /etc/profile文件
```
# ZooKeeper Env
export ZOOKEEPER_HOME=/data/app/zookeeper
export PATH=$PATH:$ZOOKEEPER_HOME/bin

source /etc/profile
```
### 启动
```
/data/app/zookeeper/bin/zkServer.sh start
```

### 集群安装注意事项
- zoo.cfg中增加节点描述
    - server.A=B:C:D
    - A: 是一个数字,表示这个是第几号服务器,
    - B: 是这个服务器的IP地址，
    - C: 第一个端口用来集群成员的信息交换,表示这个服务器与集群中的leader服务器交换信息的端口，
    - D: 是在leader挂掉时专门用来进行选举leader所用的端口。
```

server.1=192.168.199.165:2888:3888
server.2=192.168.199.184:2888:3888
server.3=192.168.199.198:2888:3888
```
- data目录中创建myid文件, 并按照以上节点描述的顺序加入此文件中
```
例如:
192.168.199.165, 此机器中的data/myid文件中的数就是1
192.168.199.184, 此机器中的data/myid文件中的数就是2
192.168.199.198, 此机器中的data/myid文件中的数就是3
```
- 如不按照此规则书写, 会如下错误:
```
java.lang.RuntimeException: My id 4 not in the peer list
```