### spring_boot_rabbitMQ
- Spring Boot集成rabbitMQ实现消息推送，rabbitMQ为异步消息处理提出了一个很好的解决方案，它是一个非常好用的消息中间件。`主要解决当生产者大量产生数据时，消费者无法快速消费的问题`。这个时候需要一个中间层，保存这个数据，rabbitMQ是一个很好的解决方案。
- Spring Boot为rabbitMQ提供了支持, Spring Boot为rabbitMQ准备了spring-boot-starter-amqp，并且为RabbitTemplate和rabbitMQ提供了自动配置选项。
- AMQP协议，即Advanced Message Queuing Protocol，高级消息队列协议，是应用层协议的一个开放标准，为面向消息的中间件设计。消息中间件主要用于组件之间的解耦，消息的发送者无需知道消息使用者的存在，反之亦然。AMQP的主要特征是面向消息、队列、路由（包括点对点和发布/订阅）、可靠性、安全。
 ### 安装rabbitMQ
- 安装brew

 `ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"`
 
 - 安装rabbitmq
 
 `brew install rabbitmq`
 
 - 启动rabbitmq server
 
 `cd /usr/local/sbin/`
 
 `sudo ./rabbitmq-server`
 
 - 成功示例，端口是5672
 
 <img width="569" alt="2018-11-06 10 13 58" src="https://user-images.githubusercontent.com/17539174/48038738-e789ea80-e1ac-11e8-9b6b-681ff9f21e16.png">
 
 ### 相关概念
- 通常我们谈到队列服务, 会有三个概念：发消息者、队列、收消息者，RabbitMQ在这个基本概念之上, 多做了一层抽象, 在发消息者和队列之间, 加入了交换器 (Exchange)。这样发消息者和队列就没有直接联系, 转而变成发消息者把消息给交换器, 交换器根据调度策略再把消息再给队列。
 
 ![rabbitmq](https://user-images.githubusercontent.com/17539174/48039609-7fd59e80-e1b0-11e8-8a08-b7dd40ed8538.png)
 
- Broker：简单来说就是消息队列服务器实体。
- Exchange：消息交换机，它指定消息按什么规则，路由到哪个队列。
- Queue：消息队列载体，每个消息都会被投入到一个或多个队列。
- Binding：绑定，它的作用就是把exchange和queue按照路由规则绑定起来。
- Routing Key：路由关键字，exchange根据这个关键字进行消息投递。
- Producer：消息生产者，就是投递消息的程序。
- Consumer：消息消费者，就是接受消息的程序。
- Channel：消息通道，在客户端的每个连接里，可建立多个channel，每个channel代表一个会话任务。

 ### 基本使用过程
 
- 客户端连接到消息队列服务器，打开一个channel。
- 客户端声明一个exchange，并设置相关属性。
- 客户端声明一个queue，并设置相关属性。
- 客户端使用routing key，在exchange和queue之间建立好绑定关系。
- 客户端投递消息到exchange。
- exchange接收到消息后，就根据消息的key和已经设置的binding，进行消息路由，将消息投递到一个或多个队列里。

 ![rabbitmq](https://user-images.githubusercontent.com/17539174/48040049-5f0e4880-e1b2-11e8-87ce-ba56f90007b5.jpg)
 
 ### Exchange四种类型
 
- Direct模式（常用）
  - Direct exchange完全根据key进行投递，只有key与绑定时的routing-key完全一致的消息才会收到消息。
- Topic模式（常用）
  - Topic exchange会根据key进行模式匹配然后进行投递，与设置的routing-key匹配上的队列才能收到消息。
- Fanout模式
  - 每个发到fanout类型交换器的消息都会分到所有绑定的队列上去。fanout交换器不处理路由键，只是简单的将队列绑定到交换器上，每个发送到交换器的消息都会被转发到与该交换器绑定的所有队列上。
- Header模式（不常用）
  - 不太常用，可以自定义匹配规则。
  
### 消息持久化
- RabbitMQ支持消息的持久化，即将消息数据持久化到磁盘上，如果消息服务器中途断开，下次开启会将持久化的消息重新发送。
- RabbitMQ的持久化有交换机、队列、消息的持久化。
  - 声明交换机Exchange的时候设置 durable=true
  
  ```
    public TopicExchange exchange() {
        return new TopicExchange(EXCHANGE_NAME,true,false);
    }
  ```
   - 声明队列Queue的时候设置 durable=true
  
  ```
    @Bean
    public Queue queue() {
        //durable：是否将队列持久化 true表示需要持久化 false表示不需要持久化
        return new Queue(QUEUE_NAME, false);
    }
  ```
     - 发送消息的时候设置消息的 deliveryMode = 2
  
  ```
    new MessageProperties() --> DEFAULT_DELIVERY_MODE = MessageDeliveryMode.PERSISTENT --> deliveryMode = 2;
  ```
    
- 出于数据安全考虑，一般消息都会进行持久化。

### 保证MQ消息的幂等性
- mq内部会为每条消息生成一个全局唯一、与业务无关的消息id，当mq接收到消息时，会先根据该id判断消息是否重复发送，mq再决定是否接收该消息。
  - Java自带的UUID
  - Twitter Snowflake
- 利用MySQL的唯一索引UNIQUE KEY，如果重复了，数据就会插入失败。

### 实例代码可以直接拉下来使用。
