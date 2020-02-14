---
layout: post
title: ActiveMQ学习笔记
comments: true
toc: true
date: 2018-02-01 20:06:02
updated: 2018-02-01 20:06:02
categories:
tags:
---
目录

    1. [X] JMS规范
    2. [X] ActiveMQ初步
    3. [X] ActiveMQAPI讲解
    4. [ ] ActiveMQ高级主题（点对点模式/发布订阅模式）
    5. [ ] 多线程+ActiveMQ负载均衡器实战

背景& JMS概述
当前CORBA，DCOM, RMI等RPC中间件技术已广泛应用与各个领域。但是面对规模和复杂都都越来约高的分布式系统，这些技术也显示出其局限性。

    1. 同步通信： 客户端发出调用后，必须等待服务对象完成处理并返回结果后才能继续执行；
    2. 客户和服务对象的声明周期紧密耦合： 客户进程和服务对象进程都必须正常运行；如果由于服务对象崩溃或者网络故障导致客户的请求不可达，客户还会接收到异常；
    3. 点对点通信：客户的一次调用只发给某个单独的目标对象。


面向消息的中间件（Message Oriented Middleware, MOM)较好的解决了以上问题。发送者将消息发给消息服务器，消息服务器将消息存放在若干队列中，在合适的时候再将消息发给接收者。这种模式下，方法送和接收是异步的，发送者无需等待；二者的声明周期未必相同：发送消息的时候接收者不一定运行，接收消息的时候发送者也不一定运行；一对多通信：对于一个消息可以有多个接收者。

Java消息服务：JMS定义了Java中范围消息中间件的接口。JMS只是接口，并没有给予实现，实现JMS接口的消息中间件成为JMS Provider, 已有的MOM系统包括Apache的ActiveMQ，以及阿里巴巴的RocketMQ，IBM的MQSeries，Microsoft的MSMQ，BEA的MesssageQ, RabbitMQ等待，他们基本都遵循JMS规范


JMS术语
JMS   实现JMS接口的消息中间件

    * Provider(MessageProducer)：生产者 -- 由会话创建的对象，用于发送消息到目标，用户可以创建某个目标的发送者，也可以创建一个通用的发送者，再发送消息时制定目标。
    * Consumer(MessageConsumer): 消费者 -- 由会话创建的对象，用于接收发送到目标的消息。消费者可以同步地（阻塞模式），或异步（非阻塞）接收队列和主题类型的消息。
    * Message  接口（消息） -- 是再消费者和生产者之间传送的对象，也就是说从一个应用程序传送到另一个应用程序，一个消息有三个主要部分：

        * 消息头（必须）： 包含黄永玉识别和为消息寻找路由的操作设置
        * 一组消息数学（可选）： 包含额外的属性， 支持其他提供者和用户的兼容。可以创建定制的字段和顾虑器（消息选择器）。
        * 一个消息体（可选）： 允许用户创建五种类型的消息（文本消息，映射消息，字节消息，流消息和对象消息）
        * 消息接口非常灵活，并提供了许多方式来定制消息的内容。
    * PTP: Point to Point， 点对点的消息模型
    * Pub/Sub: Publish/Subscribe:  发布/订阅模式
    * Queue: 队列目标
    * Topic: 主题目标
    * ConnectionFactory: 连接工厂， JMS用它创建连接：
    * Connection: JMS客户端到JMS Provider的连接
    * Destination: 消息的目的地
    * Session: 会话， 一个发送或接收请求的线程


消息格式定义
JMS定义了五种不同的消息正文格式， 以及调用的消息类型，允许你发送并接收以一些不同形式的数据， 提供现有消息格式化的一些级别的兼容性。

    * StreemMessage    Java 原始值的数据流
    * MapMessage    名称-值 对
    * TestMessage   字符串对象
    * ObjectMessage 一个序列化的Java对象
    * BytesMessage 一个未解释字节的数据流


ActiveMQ简介
ActiveMQ是Apache出品，最流行的，能力强劲的开源消息总线。
ActiveMQ是一个完全支持JMS1.1混合J2EE 1.4规范的JMS Provider实现， 尽管JMS规范初代已经是很久的事情了，但是JMS在当今的J2EE应用中仍然扮演着特殊的地位， 可以说ActiveMQ在业界应用最广泛， 当然如果想要用更强大的性能和海量数据处理能力， ActiveMQ还需要不断的升级版本， 80%以上的业务我们使用ActiveMQ已经足够满足需求，当然后续如天猫、淘宝这种大型的电商网站，尤其是双11这种特殊时间，ActiveMQ需要进行很复杂的优化原名以及架构设计才能完成，我们之后会学习一个更强大的分布式消息中间件：RocketMQ，可以说ActiveMQ是核心，是基础，所有我们必须要掌握好。

ActiveMQ Hello World
我们首先写一个简单的Hello World示例， 让大家感受下ActiveMQ, 我们需要实现接受者混合发送者两部分代码的编写。
Sender/Receiver

    1. 建立ConnectionFactory工厂对象， 需要填入用户名、密码、以及要连接的地址，均时候用默认即可，默认端口为“tcp://localhost:61616"
    2. 通过ConnectionFactory工厂我们创建一个Connection连接，并且调用Connection的start放放风开启连接，Connection默认是关闭的。
    3. 通过Connection对象创建Session会话（上下文环境对象），用于接收消息，参数配置为1为是否启用多事务， 参数配置2为签收模式，一般我们设置自动签收
    4. 通过Session创建Destination对象， 指的事一个客户端用来指定生产消息目标和消息来源的对象，在PTP模式中，Destination被称做Queue即队列，在Pub/Sub模式中，Destination被称做Topic即主题。在程序中可以使用多个Queue和Topic。
    5. 我们需要通过Session对象创建消息的发送和接收对象（生产者混合消费者）MessageProduce/MessageConsumer
    6. 我们可以使用MessageProducer的setDeliveryMode方法为其设置持久化特性和非持久化特性（DeliveryMode)
    7. 最后后我们使用JMS规范的TextMessage形式创建数据（通过Session对象）,并用MessageProducer的send放放风发送数据。同理客户段使用receive方法进行接收数据，最后不要忘记关闭Connection连接）


ActiveMQ安全机制

    * activemq的web管理界面：http://localhost:8161/admin

        * activemq管控台时候用jetty部署，要修改admin管理密码需要修改conf/jetty_realm.properites
    * activemq应该设置在又安全几张桌子，hi有符合认证的用户才能进行发送和获取消息，所以我们也可以再activemq.xml里去添加安全验证。在conf\activemq.xml，第123行规之后添加配置：

        <plugins>
          <simpleAuthenticationPlugin>
            <users>
              <authenticationUser username="hzc" password="hzc" groups="users,admins"/>
            </users>
          </simpleAuthenticationPlugin>
        </plugins>

Connection方法使用

在成功创建正确的ConnectionFactory后，下一步是创建一个连接，它是JMS定义的一个接口。ConnectionFactory负责返回可以与地层消息床底系统进行通信的Connection实现。通常客户端只使用单一连接，根据JMS文档，Connection的目的是“利用JMS提供者封装开放的连接”,以及表示“客户端与提供者服务例程之间的开发TCP/IP套接字”。该文档还指出Connection应该是进行客户端身份验证的地方等待。

当一个Connection被创建时，它的传输默认是关闭的，必须用start方法开启。一个Connection可以简历一个或多个Session.

当一个程序执行完成后，必须关闭之前创建的Connection, 否则ActiveMQ不能释放资源，关闭一个Connection同样也关闭了Session,MessageProducer和MessageConsumer

Connection createConnection();
Connection createConnection(String userName, String password, String url);

Session 方法使用

一旦从ConnectionFactory中获得一个Connection, 必须从Connection中创建一个或者多个Session.Session是一个发送或接收消息的线程，可以使用Session创建MessageProducer,MessageConsumer和Message.

Session可以被事务花，也可以不被事务化，通常，可以通过向Connection上的十点过创建方法传递一个布尔参数对此进行设置。
Session createSession(boolean transacted, int acknowledgeMode);
其中transacted为使用事务的标识，acknowledgeMode为签收模式。
结束事务由两种方法：提交或者回滚。当一个事务提交，消息被处理。如果事务中由一个步骤失败，事务就回滚，这个事务中的已经执行的动作将被测小。在发送消息最后也必须使用session.commit()方法提交事务。

签收由三种形式：

    * Session.AUTO_ACKNOWLEDGE 当客户端从receive或onMessage成功返回时，Session自动签收客户端的这条消息的收条。
    * Session.CLIENT_ACKNOWLEDGE客户端通过调用消息（Message)的acknowledge方法签收消息。在这种情况下，签收发生在Session层面：签收一个以消费的消息会自动地签收这个Session所有已消费消息的收条。
    * Session.DUPS_OK_ACKNOWLEDGE 此选项指示Session不必确保对传送消息的签收。它可能引起消息的重复，但是降低了Session的开心，所有只有客户端能容忍重复的消息，才可使用。


MessageProducer
MessageProducer: MessageProducer是一个由Session创建的对象，用来向Destination发送消息。
void send(Destionation destination, Message message);
void send(Destination destination, Message message, int deliveryMode, int priority, long time ToLive);
void send(Message message);
void send(Message message, int deliveryMode, int priority, long time ToLive);
其中deliveryMode为传送模式，priority为消息优先级， timeToLive为消息过期时间。
ActiveMQ支持两种消息传送模式，PERSSISTENT和NON_PERSISTENT两种，如果不指定传送模式，那么默认是持久性消息。如果容忍消息丢失，那么使用非持久性消息可以改善性能和减少存储的开销。
消息优先级从0-9十个级别， 0-4是普通消息，5-9是加急消息。如果不指定优先级，则默认为4，JMS不要钱严格按照这10个优先级发送消息，单必须保证加急消息要先于普通消息到达。
默认情况下，消息用不会过期，如果消息在特定周期内失去意义，那么可以设置过期时间，单位是毫秒。


