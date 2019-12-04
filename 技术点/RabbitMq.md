# 消息队列

## 消息队列基本知识

​		个人认为，消息队列是一种异步消息处理方式。所谓异步的体现，具体在于"消息"的发布和接受并不是在一次通信中完成的，在消息队列中，"消息"由发布者发布到消息队列中，消息队列作为容器，在缓存中存储消息，等待消费者来将"消息"取走。这种设计方式比起平常的http通信方式的优点是，减少了请求响应时间和解耦，主要的使用场景就是将比较耗时而且不需要即时同步返回结果的操作放入消息队列中。

​		在一些业务中，比如双十一的淘宝服务器处理订单，是可能会出现同时有很大数量的请求数，在处理这些请求的时候，是需要走很多步骤的，可能包括扣减库存、生成单据、扣除相应金额、通知商家以及顾客等操作，这些操作有一些是不需要即时返回给顾客的，比如生成单据和扣钱等操作，这些操作也就没有必要即时完成。在这个时候如果用同步的消息处理方式，会给服务器造成很大的压力，所以使用到消息队列可以将这些请求先放进缓存的消息队列中，然后服务器这边对消息队列中的消息进行定时提取进行操作。

​		上述举例是为了说明消息队列的减少请求响应时间和解耦的作用。

## RabbitMq消息队列简介

​		 RabbitMQ 是一个由 Erlang 语言开发的 AMQP 的开源实现。 

​		AMQP ：Advanced Message Queue，高级消息队列协议。它是应用层协议的一个开放标准，为面向消息的中间件设计，基于此协议的客户端与消息中间件可传递消息，并不受产品、开发语言等条件的限制。

​		RabbitMQ 最初起源于金融系统，用于在分布式系统中存储转发消息，在易用性、扩展性、高可用性等方面表现不俗。具体特点包括：

1. 可靠性（Reliability）
    RabbitMQ 使用一些机制来保证可靠性，如持久化、传输确认、发布确认。
2. 灵活的路由（Flexible Routing）
    在消息进入队列之前，通过 Exchange 来路由消息的。对于典型的路由功能，RabbitMQ 已经提供了一些内置的 Exchange 来实现。针对更复杂的路由功能，可以将多个 Exchange 绑定在一起，也通过插件机制实现自己的 Exchange 。
3. 消息集群（Clustering）
    多个 RabbitMQ 服务器可以组成一个集群，形成一个逻辑 Broker 。
4. 高可用（Highly Available Queues）
    队列可以在集群中的机器上进行镜像，使得在部分节点出问题的情况下队列仍然可用。
5. 多种协议（Multi-protocol）
    RabbitMQ 支持多种消息队列协议，比如 STOMP、MQTT 等等。
6. 多语言客户端（Many Clients）
    RabbitMQ 几乎支持所有常用语言，比如 Java、.NET、Ruby 等等。
7. 管理界面（Management UI）
    RabbitMQ 提供了一个易用的用户界面，使得用户可以监控和管理消息 Broker 的许多方面。
8. 跟踪机制（Tracing）
    如果消息异常，RabbitMQ 提供了消息跟踪机制，使用者可以找出发生了什么。
9. 插件机制（Plugin System）
    RabbitMQ 提供了许多插件，来从多方面进行扩展，也可以编写自己的插件。

## RabbitMq概念模型

​		所有 MQ 产品从模型抽象上来说都是一样的过程：

​		消费者（consumer）订阅某个队列。生产者（producer）创建消息，然后发布到队列（queue）中，最后将消息发送到监听的消费者。 

1.Message

消息，消息是不具名的，它由消息头和消息体组成。消息体是不透明的，而消息头则由一系列的可选属性组成，这些属性包括routing-key（路由键）、priority（相对于其他消息的优先权）、delivery-mode（指出该消息可能需要持久性存储）等。

2.Publisher

 消息的生产者，也是一个向交换器发布消息的客户端应用程序。

3.Exchange

交换器，用来接收生产者发送的消息并将这些消息路由给服务器中的队列。

4.Binding

绑定，用于消息队列和交换器之间的关联。一个绑定就是基于路由键将交换器和消息队列连接起来的路由规则，所以可以将交换器理解成一个由绑定构成的路由表。

5.Queue

消息队列，用来保存消息直到发送给消费者。它是消息的容器，也是消息的终点。一个消息可投入一个或多个队列。消息一直在队列里面，等待消费者连接到这个队列将其取走。

6.Connection

网络连接，比如一个TCP连接。

7.Channel

信道，多路复用连接中的一条独立的双向数据流通道。信道是建立在真实的TCP连接内地虚拟连接，AMQP 命令都是通过信道发出去的，不管是发布消息、订阅队列还是接收消息，这些动作都是通过信道完成。因为对于操作系统来说建立和销毁 TCP 都是非常昂贵的开销，所以引入了信道的概念，以复用一条 TCP 连接。

8.Consumer

消息的消费者，表示一个从消息队列中取得消息的客户端应用程序。

9.Virtual Host

虚拟主机，表示一批交换器、消息队列和相关对象。虚拟主机是共享相同的身份认证和加密环境的独立服务器域。每个 vhost 本质上就是一个 mini 版的 RabbitMQ 服务器，拥有自己的队列、交换器、绑定和权限机制。vhost 是 AMQP 概念的基础，必须在连接时指定，RabbitMQ 默认的 vhost 是 / 。

10.Broker

表示消息队列服务器实体。

## RabbitMq 安装部署

RabbitMq的安装部署下面说了三种，目前建议在公司内网机开发时使用windows系统下的安装，可以进行调试。当项目上线的时候可以在外网机搭建虚拟机，然后在docker中部署，将docker中的镜像包保存到本地然后发送到内网内测服务器上进行docker上的部署。

### Windows

先安装[Erlang](http://www.erlang.org/download/otp_win64_17.3.exe "Erlang"),然后再安装[Rabbitmq](http://www.rabbitmq.com/ "RabbitMq")。

Erlang属于RabbitMq运行的环境，两者在windows系统下安装都比较简单，按照安装界面的步骤一步步进行就可以了，下面讲一下安装好了之后的操作。

安装好RabbitMq之后，win + r 搜索RabbitMQ Command Prompt并以管理员身份运行。

在命令行中输入以下命令：

启动RabbitMq的可视化工具

```
rabbitmq-plugins enable rabbitmq_management
```

启动RabbitMq的日志(推荐，可用于开发调试和运维)

```
rabbitmq-plugins enable rabbitmq_tracing
```

### Linux

同样是安装Erlang环境然后安装RabbitMq

```
sudo apt-get install erlang-nox 
sudo apt-get update 
sudo apt-get install rabbitmq-server
```

### Docker

Docker中部署比较简单，不需要安装环境，只需要安装镜像源就可以了。

```
docker search rabbitmq:management
docker pull rabbitmq:management
```

启动镜像

```
docker run -d -p 5672:5672 -p 15672:15672 --name rabbitmq rabbitmq:management
```

进入docker中的消息队列容器中，用上面windows安装方法中的命令启动相应插件。

## RabbitMq五种消息队列

1.简单模式：一个生产者P发送消息到队列Q,一个消费者C接收

2.工作队列模式：一个生产者，多个消费者，每个消费者获取到的消息唯一，多个消费者只有一个队列。

3.发布订阅模式：一个生产者发送的消息会被多个消费者获取。一个生产者、一个交换机、多个队列、多个消费者。

4.路由模式：生产者发送消息到交换机并且要指定路由key，消费者将队列绑定到交换机时需要指定路由key。

5.通配符模式Topic：生产者P发送消息到交换机X，type=topic，交换机根据绑定队列的routing key的值进行通配符匹配。

​		以下给出c#实现RabbitMq的路由模式的消息队列代码。

```c#
public static void Routing()
{
    var factory = new ConnectionFactory();
    factory.HostName = "localhost";
    factory.UserName = "guest";
    factory.Password = "guest";

    using (var connection = factory.CreateConnection())
    {
        using (var channel = connection.CreateModel())
        {
            // 声明一个交换机
            channel.ExchangeDeclare("test_exchange", ExchangeType.Direct, true, false, null);
            string message = "RoutingTest";
            var body = Encoding.UTF8.GetBytes(message);
            channel.BasicPublish(exchange_name, routing_key, null, body);
            Console.WriteLine("已发送: {0}", message);
            Console.ReadLine();
        }
    }
}
```

​		上述是消息队列的发送的代码

```c#
public static List<string> Routing()
{
    var factory = new ConnectionFactory();
    factory.AutomaticRecoveryEnabled = true;
    factory.HostName = "localhost";
    factory.UserName = "guest";
    factory.Password = "guest";

    string message = "";
    List<string> list = new List<string>();

    using (var connection = factory.CreateConnection())
    {
        using (var channel = connection.CreateModel())
        {
            channel.QueueDeclare(queue_name, false, false, false, null);

            channel.QueueBind(queue_name, exchange_name, routing_key);

            var consumer = new EventingBasicConsumer(channel);
            channel.BasicConsume(queue_name, false, consumer);
            consumer.Received += (model, ea) =>
            {
                var body = ea.Body;
                message = Encoding.UTF8.GetString(body);
                list.Add(message);
                Console.WriteLine("已接收： {0}", message);
                channel.BasicAck(ea.DeliveryTag, false);
            };
            Console.ReadLine();
        }
    }
    return list;
}
```

​		上述是消息队列的消费者的代码

## 消息队列消费订阅方式

​		消息队列的消费订阅方式有两种：一种是push，消息队列服务器主动推送给消费者，这里需要消费者一直监听服务器，保持接受服务器的消息的状态，消息的推送与否决定权在于服务器，这种方式对于消息处理的解耦十分出色，更加贴近于发布订阅模式，不会给消息队列服务器造成很大的压力，也真正让消息的推送和处理分开了；另一种是pull，主动权到了消费者手中，消费者决定何时去消息队列服务器中获取消息，一般常见的具体实现方式有网页端的ajax轮询和服务端的定时任务处理，这种方式对服务端的压力比较小，但是可能会给消息队列服务器一定的压力，消息作为数据不被提取是会在服务器上一直缓存，而且在这种方式中服务端只是把消息队列服务器当做了一个存储数据的地方，

​		上述两种方法中，push方法适用于消息量小，不需要服务器承担很大的压力的使用场景，消息的处理集中在服务端，但是在服务端故障的情况下，无效的push会对服务端有一定负载，不能保证服务端处理消息成功；pull方法适用于消息量大，给服务端的任务相对轻，可以在分布式的服务器上分别处理消息，但是服务端对消息队列服务器的请求很多时候可能是无效的，所以会浪费带宽和服务器处理能力。

## 消息队列远程连接方法

​		消息队列安装过后，默认的host是localhost，如果要在别的主机上对消息队列服务器进行远程连接，需要对administrator用户进行新的权限设置，或者是建立一个新的用户给别的主机访问。

```
rabbitmqctl add_user username password
rabbitmqctl set_user_tags username administrator
rabbitmqctl set_permissions -p username ".*" ".*" ".*"
```

