# 为什么选择RabbitMQ

现在的市面上有很多MQ可以选择，比如ActiveMQ、ZeroMQ、Appche Qpid，那问题来了为什么要选择RabbitMQ

    1.除了Qpid，RabbitMQ是唯一一个实现了AMQP标准的消息服务器；
    2.可靠性，RabbitMQ的持久化支持，保证了消息的稳定性；
    3.高并发，RabbitMQ使用了Erlang开发语言，Erlang是为电话交换机开发的语言，天生自带高并发光环，和高可用特性；
    4.集群部署简单，正是应为Erlang使得RabbitMQ集群部署变的超级简单；
    5.社区活跃度高，根据网上资料来看，RabbitMQ也是首选；

## 工作机制

- 生产者：消息的创建者，负责创建和推送数据到消息服务器
- 消费者：消息的接收方，用于处理数据和确认消息
- 代理：就是RabbitMQ本身，用于扮演“快递”的角色，本身不生产消息，只是扮演“快递”的角色

## 消息发送原理

首先你必须连接到Rabbit才能发布和消费消息，那怎么连接和发送消息的呢？
你的应用程序和Rabbit Server之间会创建一个TCP连接，一旦TCP打开，并通过了认证，
认证就是你试图连接Rabbit之前发送的Rabbit服务器连接信息和用户名和密码，有点像程序连接数据库，
一旦认证通过,你的应用程序和Rabbit就创建了一条AMQP信道（Channel）。
信道是创建在“真实”TCP上的虚拟连接，AMQP命令都是通过信道发送出去的，每个信道都会有一个唯一的ID，
不论是发布消息，订阅队列或者介绍消息都是通过信道完成的。

为什么不通过TCP直接发送命令？
对于操作系统来说创建和销毁TCP会话是非常昂贵的开销，假设高峰期每秒有成千上万条连接，
每个连接都要创建一条TCP会话，这就造成了TCP连接的巨大浪费，而且操作系统每秒能创建的TCP也是有限的，
因此很快就会遇到系统瓶颈。

## 组件

- ConnectionFactory（连接管理器）：应用程序与Rabbit之间建立连接的管理器，程序代码中使用
- Channel（信道）：消息推送使用的通道
- Exchange（交换器）：用于接受、分配消息
- Queue（队列）：用于存储生产者的消息
- RoutingKey（路由键）：用于把生成者的数据分配到交换器上
- BindingKey（绑定键）：用于把交换器的消息绑定到队列上

 ![image](https://gitee.com/heguangchuan/rainmeter/raw/master/img/rabbitmq/yuanli.png)

### Message

消息头(routing-key,priority,delivery-mode) + 消息体

### Publisher

消息生产者  send message to exchange

### Exchange

接收生产者消息 并路由给队列

    direct
    fanout
    topic
    headers

### Queue

保存消息 直到发送给消费者

### Binding

消息队列与交换机的关联

### Connection

网络连接

### Channel

信道

### Consumer

消息的消费者

### Virtual Host

虚拟主机

### Broker

消息队列服务器实体

### 消息持久化

Rabbit队列和交换器有一个不可告人的秘密，就是默认情况下重启服务器会导致消息丢失，那么怎么保证Rabbit在重启的时候不丢失呢？答案就是消息持久化。当你把消息发送到Rabbit服务器的时候，你需要选择你是否要进行持久化，但这并不能保证Rabbit能从崩溃中恢复，想要Rabbit消息能恢复必须满足3个条件

- 投递消息的时候durable设置为true，消息持久化
- 消息已经到达持久化交换器上
- 消息已经到达持久化的队列

### 持久化工作原理

Rabbit会将你的持久化消息写入磁盘上的持久化日志文件，等消息被消费之后，Rabbit会把这条消息标识为等待垃圾回收。
    

消息持久化的优点显而易见，但缺点也很明显，那就是性能，因为要写入硬盘要比写入内存性能较低很多，从而降低了服务器的吞吐量，尽管使用SSD硬盘可以使事情得到缓解，但他仍然吸干了Rabbit的性能，当消息成千上万条要写入磁盘的时候，性能是很低的。所以使用者要根据自己的情况，选择适合自己的方式

### 虚拟主机

每个Rabbit都能创建很多vhost，我们称之为虚拟主机，每个虚拟主机其实都是mini版的RabbitMQ，拥有自己的队列，交换器和绑定，拥有自己的权限机制

### 安装

    mkdir -p /usr/local/rabbitmq/conf
    mkdir -p /usr/local/rabbitmq/data

    docker run --name rabbitmq --restart=always -p 5672:5672 -p 15672:15672 -d rabbitmq:3.7.7-management
    docker cp rabbitmq:/etc/rabbitmq /usr/local/rabbitmq/conf
    docker rm -f rabbitmq 
    docker run --name rabbitmq --restart=always \
    -p 5672:5672 -p 15672:15672 \
    -v /usr/local/rabbitmq/conf:/etc/rabbitmq \
    -v /usr/local/rabbitmq/data:/var/lib/rabbitmq -d rabbitmq:3.7.7-management



​    