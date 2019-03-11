# Zookeeper

## 1. Zookeeper 安装

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
### 启动Zookeeper服务
```
/data/app/zookeeper/bin/zkServer.sh start
```
### 客户端连接
在zookeeper/bin目录中, zkCli.sh为客户端启动命令, 具体如下:
```
./zkCli.sh -timeout 5000 -server 192.9.200.242:2181
- timeout: 定义客户端和服务端的心跳间隔为5000, 单位为毫秒, 如果超出5秒没有收到心跳,服务端认为此连接失效.
- server: IP:port, 
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

## 2. zookeeper 名词

### 2.1 znode(节点)
每个子目录项被称作为znode，和文件系统一样，我们能够自由的增加、删除znode，在一个znode下增加、删除子znode，唯一的不同在于znode是可以存储数据的。 
#### 2.1.1 znode 分类
- PERSISTENT: 持久化节点, 在zookeeper连接断开后, 节点数据依然存在.
- PERSISTENT_SEQUENTIAL: zookeeper给该类型的节点进行顺序编号, 连接断开后, 节点数据依然存在.
- EPHEMERAL: 临时目录节点, 在zookeeper连接断开后, 节点数据删除.
- EPHEMERAL_SEQUENTIAL: zookeeper给该类型的节点进行顺序编号,临时目录节点, 在zookeeper连接断开后, 节点数据删除.

### 2.2 连接
每一个client都是以长连接的方式连接Zookeeper Server

## zookeeper 机制
### 选举机制
