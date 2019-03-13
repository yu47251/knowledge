# Zookeeper

## 1. Zookeeper 安装
### 下载
```
wget https://mirrors.tuna.tsinghua.edu.cn/apache/zookeeper/zookeeper-3.5.4-beta/zookeeper-3.5.4-beta.tar.gz
```
### 解压
```
sudo tar -zxvf zookeeper-3.5.4.tar.gz
```
### 目录重命名
```
sudo mv zookeeper-3.5.4 zookeeper
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
每一个client都是以长连接的方式连接Server.

### 2.3 选举

### 2.4 顺序一致性

## 3. 命令

## 4. spring集成zookeeper
- 注意: spring依赖curator, curator对zookeeperClient进行封装. curator的jar包4.0以上版本需要zookeeper3.5的版本. curator4.0版本对应的zookeeper3.5以下.
- 以下代码中均是以zookeeper3.5为演示. 目前(2019-03-13)zookeeper3.5为beta版本, 请慎用.
### 4.1 spring-boot集成zookeeper
### 4.2 spring-cloud集成zookeeper作为服务注册中心
### 4.3 spring-cloud集成zookeeper作为配置管理
- 实现服务配置集中管理, zkUI修改properties, 可以直接生效.
#### 4.3.1 pom.xml配置文件
```xml
    <properties>
        <java.version>1.8</java.version>
        <spring-cloud.version>Greenwich.RC2</spring-cloud.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-zookeeper-all</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-config</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <optional>true</optional>
        </dependency>
    </dependencies>
```
#### 4.3.2 bootstrap.yml配置文件
```yml
spring:
  application:
    name: order-service
  cloud:
    zookeeper:
      connect-string: 192.168.56.102:2181,192.168.56.103:2181,192.168.56.104:2181
      // 最大重试次数
      max-retries: 10
      // 最大sleep时间
      max-sleep-ms: 50
      block-until-connected-wait: 500
      block-until-connected-unit: milliseconds
      // 设置使用zookeeper
      enabled: true
      // 服务发现相关配置
      discovery:
        // 是否启用zookeeper作为服务发现
        enabled: true
        // 是否启用zookeeper作为服务注册
        register: true
        // 注册后当前实例的服务访问IP
        instance-host: 192.168.56.1
        // 注册后当前实例的服务访问端口
        instance-port: ${server.port}
        root: services
      // 基于zookeeper的配置管理配置项, 
      // 按.提示没有, 需要找类ZookeeperConfigProperties,按照类中的属性,进行设置
      config:
        // 自定义配置管理在zookeeper中的路径
        root: /configuration
        enabled: true
        // 启动zookeeper配置中心监听, 修改会有event
        watcher:
          enabled: true
    // 使 cloud 中的默认config服务失效, 默认是生效, 调用本地的8888端口的config服务      
    config:
      enabled: false
```
#### 4.3.3 java代码
- 启动类
```java
@EnableDiscoveryClient
@SpringBootApplication
public class OrderServiceApplication {
    public static void main(String[] args) {
        new SpringApplicationBuilder(OrderServiceApplication.class)
                .web(WebApplicationType.SERVLET).run(args);
    }
}
```
- 服务实现
```java
/**
 * zookeeper配置中心加载
 */
@RefreshScope
@Service
public class OrderServiceImpl implements OrderService {
    // zookeeper中修改foo的value, 可以及时生效, @RefreshScope
    @Value("${foo}")
    private String remark;

    @Override
    public Order getInfo(){
        Order order = new Order();
        order.setRemark(remark);
        return order;
    }
}
```