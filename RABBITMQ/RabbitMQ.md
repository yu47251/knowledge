# RabbitMQ
## 1. 安装
- 以下安装流程为学习使用，为最简单的安装方式，如需HA，请令行google，谢谢
- 由于RabbitMQ底层是由ErLang实现，所以需要安装Erlang语言包，在安装Erlang包前，需要安装依赖
```
yum -y install make gcc gcc-c++ kernel-devel m4 ncurses-devel openssl-devel
```
### 1.1 Erlang 下载，解压，安装，配置

下载链接地址，请安装官网最新文档版本，官网地址：http://www.erlang.org/downloads，本文使用版本：OTP 21.2
```
wget http://erlang.org/download/otp_src_21.2.tar.gz
tar -zxvf otp_src_21.2.tar.gz
cd otp_src_21.2
./configure --prefix=/usr/local/erlang --with-ssl --enable-threads --enable-smp-support --enable-kernel-poll --enable-hipe --without-javac
make & make install
```
### 1.2 RabbitMQ安装
- 下载地址
```
wget http://www.rabbitmq.com/releases/rabbitmq-server/v3.6.15/rabbitmq-server-generic-unix-3.6.15.tar.xz
```
- 解压
```
tar xvf rabbitmq-server-generic-unix-3.6.15.tar.xz
mv rabbitmq_server-3.6.15 /usr/local/RabbitMQ
```
- 设置环境变量
```
# vim /etc/profile
在末尾加入以下内容：
#set RabbitMQ environment
export PAHT=$PATH:/usr/local/RabbitMQ/sbin
```
- 使环境变量生效
```
source /etc/profile
```
- 查看rabbitmq服务提供的插件
```
# ./rabbitmq-plugins list
```
- 启动管理界面
```
# ./rabbitmq-plugins enable rabbitmq_management
```

## 2 spring data amqp 

### 2.1 延迟消息

安装插件rabbitmq_delayed_message_exchange

#### 2.1.1 rabbitConfig
```java
@Configuration
public class QueueConfig {

    @Bean
    public CustomExchange delayExchange() {
        Map<String, Object> args = new HashMap<>();
        args.put("x-delayed-type", "direct");
        return new CustomExchange("test_exchange", "x-delayed-message",true, false,args);
    }

    @Bean
    public Queue queue() {
        Queue queue = new Queue("test_queue_1", true);
        return queue;
    }

    @Bean
    public Binding binding() {
        return BindingBuilder.bind(queue()).to(delayExchange()).with("test_queue_1").noargs();
    }
}
```
#### 2.1.2 发送
```java
@Service
public class MessageServiceImpl {

    @Autowired
    private RabbitTemplate rabbitTemplate;

    public void sendMsg(String queueName,String msg) {
        SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
        System.out.println("消息发送时间:"+sdf.format(new Date()));
        rabbitTemplate.convertAndSend("test_exchange", queueName, msg, new MessagePostProcessor() {
            @Override
            public Message postProcessMessage(Message message) throws AmqpException {
                message.getMessageProperties().setHeader("x-delay",3000);
                return message;
            }
        });
    }
}
```